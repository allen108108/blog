---
title: "OpenClaw：同一 Provider 的 Auth Profiles 輪換（Rotation / Failover）——從「為什麼」到「怎麼做」的生產級指南"
date: 2026-02-24 13:16:00
categories:
  - OpenClaw
  - Reliability
tags:
  - OpenClaw
  - Auth
  - Profile
  - Rotation
  - Failover
  - LLM
  - Provider
---

## 導言：你以為你在做 Model Fallback，其實你先死在「同一個 Provider 的同一個帳號」

我很常看到一個典型失敗情境：

- 你以為自己已經把 `agents.defaults.model.fallbacks` 配好了：主力模型掛了就換下一個。
- 結果實際上你遇到的不是「模型掛了」，而是：
  - **速率限制 (rate limit)**
  - **網路抖動 / timeout**
  - **某個帳號額度用完 (out-of-quota / billing)**
  - **某個 OAuth token 過期 (auth error)**

這些錯誤通常發生在「同一 provider」的層級，而且往往只跟**某一組憑證**有關。

如果你把所有容錯都押在「跨模型 fallback」，你會付出更高延遲、更高不確定性，甚至更高成本（因為你可能把一個暫時性 rate limit，升級成跨供應商切換）。

OpenClaw 的設計其實更像工程系統的兩層保護：

1. **Rotation / Failover（同一 Provider 內：多個 auth profiles）**
2. **Model Fallback（跨 models / providers）**

這篇文章要做的事情，就是把這兩層講清楚，並教你怎麼「可控地」使用它。

---

## 目錄（TOC）

- [1. 一句話總結：rotation 與 fallback 的邊界](#1-一句話總結rotation-與-fallback-的邊界)
- [2. 名詞與心理模型：先把抽象名詞變成可操作物件](#2-名詞與心理模型先把抽象名詞變成可操作物件)
- [3. 底層邏輯（Why）：為何需要 session 黏性 (stickiness)？](#3-底層邏輯why為何需要-session-黏性-stickiness)
- [4. 視覺化架構（Visual Architecture）：rotation → fallback 一張圖看懂](#4-視覺化架構visual-architecturerotation--fallback-一張圖看懂)
- [5. 技術解構（What）：Profile 從哪裡來、放哪裡、怎麼排序](#5-技術解構whatprofile-從哪裡來放哪裡怎麼排序)
- [6. 實戰演練（How）：新增第二個 profile、釘住 profile、觀測與驗證](#6-實戰演練how新增第二個-profile釘住-profile觀測與驗證)
- [7. 進階維運（Advanced Ops）：錯誤分級、退避策略、成本與多 agent 一致性](#7-進階維運advanced-ops錯誤分級退避策略成本與多-agent-一致性)
- [8. 排障手冊：看起來「新增 profile 但沒被用到」怎麼辦？](#8-排障手冊看起來新增-profile-但沒被用到怎麼辦)
- [9. 後記：把它當成 SRE 的路由層來設計](#9-後記把它當成-sre-的路由層來設計)
- [參考資料](#參考資料)
- [授權](#授權)

---

## 1. 一句話總結：rotation 與 fallback 的邊界

**OpenClaw 會先在同一個 provider 內輪換 (rotation) 不同 auth profiles；只有當該 provider 的 profiles 都不可用，才會進入跨模型的 fallback。**[^1]

> 你可以把它想成：
> - rotation = 同一家電信商換不同 SIM 卡
> - fallback = 換另一家電信商

你通常應該先嘗試「同一家電信商」的容錯，因為切換成本更低、行為更可預測。

---

## 2. 名詞與心理模型：先把抽象名詞變成可操作物件

這裡把文獻中的關鍵名詞用「你真的會操作的東西」來解釋：

- **Provider**：模型供應商/驅動（例如 `openai-codex`、`anthropic`、`google`）。[^1]
- **Auth profile**：同一 provider 的「一組憑證」（OAuth 或 API key）。[^1]
- **Profile ID**：OpenClaw 用來識別一個 profile 的字串，例如：
  - `openai-codex:default`
  - `openai-codex:you@example.com`（如果 provider/OAuth 能拿到 email）[^1]
- **Rotation / Failover**：同 provider 內，某 profile 失敗 → 立即換下一個 profile。[^1]
- **Fallback**：同 provider 的 profiles 全部不可用 → 才換 `agents.defaults.model.fallbacks` 的下一個模型。[^1]

一句話心理模型：

> **Model** 決定你要去哪一家供應商買服務；**Profile** 決定你用哪張會員卡刷卡。

---

## 3. 底層邏輯（Why）：為何需要 session 黏性 (stickiness)？

文件裡提到 OpenClaw 的 **session 黏性（stickiness）**：同一個 session 通常會固定使用同一個 profile，而不是每次 request 都輪替。[^1]

這是一體兩面的事情。

### 3.1 直覺好處：穩定、快取、可重現

如果每次 request 都輪替 profile，你會遇到：

- **快取命中率下降**：同一 session 內的「上下文與行為」變得飄。
- **除錯變困難**：你很難重現「剛剛那次為什麼失敗」。
- **成本不可預測**：不同帳號/不同 key 可能對應不同的計費方案。

黏性就是用「固定路徑」換「可預期性」。

### 3.2 代價：你會覺得系統「怎麼不幫我自動換？」

黏性導致一個常見誤會：

- 你新增了第二個 profile
- 但你還在同一個 session
- 看起來「系統沒有在用新 profile」

其實它是在遵守黏性策略——除非觸發某些事件，才會改選下一個 profile。[^1]

### 3.3 什麼時候會打破黏性？

文件列出幾種常見情境：[^1]

- 你開始新 session（例如 `/new`、`/reset`）
- session 壓縮完成（compression 計數增加）
- 當前 profile 進入 cooldown / disabled
- 當前 profile 本次呼叫出錯，觸發 failover

把它想成「路由表不是每秒重算」：只有在必要時才變。

---

## 4. 視覺化架構（Visual Architecture）：rotation → fallback 一張圖看懂

以下架構圖改寫自原始文獻，保留重點節點（rotation 與 fallback 的邊界）。[^1]

```plain text
+------------------+
|      請求         |
+------------------+
         |
         v
+------------------------------+
| 選擇模型（primary）           |
+------------------------------+
         |
         v
+----------------------------------------------+
| 選擇 Auth Profile（同 session 黏性 sticky）    |
+----------------------------------------------+
         |
         v
+----------------------------------------------+
| 以該 profile 呼叫 provider                    |
+----------------------------------------------+
    |                     |
    | 成功                | 失敗
    v                     v
+-----------+     +----------------------------+
|   完成    |     | 錯誤分類                   |
+-----------+     +----------------------------+
                          |
          +---------------+------------------+
          |               |                  |
          v               v                  v
+----------------+  +------------------+  +----------------------+
| 速率限制 /     |  | 額度不足 /        |  | 認證錯誤             |
| timeout        |  | out-of-quota /    |  |（可能需重登）        |
|                |  | billing           |  |                      |
+----------------+  +------------------+  +----------------------+
        |                 |                     |
        v                 v                     v
+----------------+  +------------------+  +----------------------+
| 標記 COOLDOWN  |  | 標記 DISABLED    |  | 嘗試下一個 profile   |
|（短期退避）     |  |（長期退避）       |  |（或提示重登）        |
+----------------+  +------------------+  +----------------------+
```

（續圖：round-robin 與最終 model fallback）

```plain text
        |                 |
        +--------+--------+
                 |
                 v
+----------------------------------------------+
| 嘗試下一個 profile（round-robin；跳過壞狀態）  |
+----------------------------------------------+
                 |
                 v
+----------------------------------------------+
| 該 provider 所有 profiles 都不可用？          |
+----------------------------------------------+
        | 否                          | 是
        v                             v
（回到呼叫）     +--------------------------------------------+
                 | Model fallback -> agents.defaults.model...  |
                 +--------------------------------------------+
```

你應該從這張圖得到兩個工程直覺：

1. rotation 是常態、fallback 是「最後手段」
2. 錯誤分類決定 profile 的狀態機（cooldown / disabled / next）

---

## 5. 技術解構（What）：Profile 從哪裡來、放哪裡、怎麼排序

### 5.1 Profiles 存放位置：按 agent 分開

文獻給出關鍵資訊：profile 的密鑰/OAuth token 存在每個 agent 自己的檔案裡：[^1]

- `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

並且：

- **多個 agent 可能有多份 `auth-profiles.json`**
- **主 agent 的憑證不會自動共享給其他 agent**[^1]

這點對使用 `sessions_spawn` 或多 agent pipeline 的人很重要：你可能在主 agent 跑得很順，子 agent 卻因為沒有 profile 而直接炸掉。

### 5.2 Profile 選擇順序（Order）：三層優先序

文獻把選擇邏輯濃縮成三個來源（由高到低）：[^1]

1. 你顯式指定的 `auth.order[provider]`
2. gateway config 的 `auth.profiles` 中屬於該 provider 的 profiles
3. 該 agent 的 `auth-profiles.json` 中已存的 profiles

如果你沒有指定順序，OpenClaw 會採「輪詢 + 健康狀態」的策略（摘要如下）：[^1]

- OAuth profiles 通常優先於 API key
- 傾向挑「較久沒用」的 profile（分散使用量）
- 冷卻/禁用的 profile 會被放到後面（甚至跳過）

### 5.3 Cooldown vs Disabled：把暫時性與非暫時性故障分流

這段設計我特別喜歡，因為它很符合 SRE 的思維：

- **Cooldown（短期退避）**：暫時性錯誤，例如 rate limit / timeout / 網路抖動。通常指數退避，例如 `1m → 5m → 25m → 1h`。[^1]
- **Disabled（較長退避）**：額度/計費錯誤，例如 out-of-quota、credits 不足。退避可能數小時到 24h，期間 round-robin 會直接跳過它。[^1]

這兩者的差異是：

> cooldown 假設「它會自己好」；disabled 假設「需要人介入」。

---

## 6. 實戰演練（How）：新增第二個 profile、釘住 profile、觀測與驗證

> 注意：以下指令為文獻內容整理；實際支援的 flags 以你環境中的 `openclaw ... --help` 為準。[^1]

### 6.1 新增同一 provider 的另一個 profile（OAuth 類）

最直接的做法是「再跑一次 OAuth 流程」，就會新增另一個 profile：[^1]

```bash
# 重新跑一次 OAuth 以新增另一個帳號（會新增新的 profile）
openclaw models auth login --provider openai-codex

# 或走 onboarding 向導
openclaw onboard --auth-choice openai-codex
```

如果 provider 能拿到 email，你可能會得到類似：

- `openai-codex:you@example.com`

如果拿不到，可能仍是：

- `openai-codex:default`

### 6.2 新增 profile（API key 類）

某些 provider 支援以 API key 新增 profile：[^1]

```bash
openclaw models auth add
```

文獻也提醒：部分流程可能支援 `--profile-id <provider:xxx>`，如果你希望自訂命名（如 `anthropic:work` / `anthropic:personal`），建議查看 provider 文件或 `openclaw models auth login --help`。[^1]

### 6.3 強制釘住特定 profile：避免自動輪換

這在「成本控制」或「debug 可重現」時很有用。

#### 方式 A：設定 `auth.order[provider]`

在 gateway config 設定你希望優先使用的 profile：[^1]

```jsonc
{
  "auth": {
    "order": {
      "openai-codex": ["openai-codex:default"]
    }
  }
}
```

> 工程含義：你在同一個 provider 內建立一個穩定的優先序；如果你只填一個 profile，某種程度上就是「固定只用它」。

#### 方式 B：每個 session 指定 profile（若 UI 支援）

文獻提到部分介面支援這種格式：[^1]

- `/model openai-codex/gpt-5.2@openai-codex:default`

它會把 session 釘在指定 profile 上，直到你開始新的 session。

### 6.4 觀測：我到底用的是哪個 model / 哪個 profile？

文獻提供的最直接檢查方式：[^1]

```bash
openclaw models status
```

我會把它當成「現場儀表板」：

- 用來確認 rotation 有沒有發生
- 用來確認 `auth.order` 是否生效
- 用來確認某 profile 是否進入 cooldown/disabled

---

## 7. 進階維運（Advanced Ops）：錯誤分級、退避策略、成本與多 agent 一致性

這一節是我把文獻內容「往工程體系抬升」的地方：如果你只是個人使用，可能用不到全部；但只要你開始在意穩定性/成本/團隊協作，這些會變成你的護城河。

### 7.1 錯誤分級（Error Taxonomy）就是你的路由策略

文獻列出會觸發 rotation 的典型錯誤：[^1]

- rate limit（暫時性）→ cooldown → 換下一個 profile
- timeout/連線錯誤（暫時性）→ cooldown → 換下一個 profile
- auth errors（可能需重登）→ 能換就換，不行就提示介入
- out-of-quota/billing（非暫時性）→ disabled（長退避）→ 跳過

如果你把這套策略落地到團隊，你可以制定一個「SLO 友善」的處置準則：

- 看到 429 / timeout 這種：**不要急著換 provider**（先 rotation）
- 看到 billing/out-of-quota：**立刻停撞牆**（disabled + 告警）

### 7.2 成本控制：把 profiles 當成「金/銀/銅 tier」

你可以用 `auth.order` 做成本分層：

- 便宜但不穩的帳號/profile（先用）
- 貴但穩的帳號/profile（當備援）

搭配 model fallback，你就得到三層保護：

1. 同 provider 多 profiles（最低切換成本）
2. 同 provider 內「更貴但更穩」profile（次級成本）
3. 跨 provider fallback（最高切換成本，但最能救火）

### 7.3 多 agent 一致性：你必須明確「誰擁有哪些 profiles」

因為 `auth-profiles.json` 是按 agent 存放的：[^1]

- 主 agent 有的 profile，子 agent 不一定有

如果你常用 `sessions_spawn`，我會建議你建立一個簡單規範：

- 哪些 provider 一定要在所有 agent 登入
- 哪些 provider 只允許在主 agent 登入（避免成本失控）

這是「權限治理」的一部分，而不是純技術問題。

---

## 8. 排障手冊：看起來「新增 profile 但沒被用到」怎麼辦？

文獻給了非常實用的排查方向，我把它整理成一個順序（你可以當 checklist）：[^1]

1. **你是不是還在同一個 session？**
   - 黏性會讓你一直用舊 profile，除非你開新 session 或觸發 failover。
2. **你有沒有設定 `auth.order`？**
   - 如果你把新 profile 排在後面，你永遠「看起來」用不到它。
3. **舊 profile 是否其實還沒進 cooldown/disabled？**
   - 沒出錯就不換，是預期行為。
4. **看 `auth-profiles.json`：新 profile 真的寫進去了嗎？**
   - 檔案按 agent 分開，別看錯地方。
5. **用 `openclaw models status` 確認當前 model/profile**

---

## 9. 後記：把它當成 SRE 的路由層來設計

我會用一句話收斂：

> **rotation 是把「個別憑證」的脆弱性隔離掉；fallback 是把「供應商」的脆弱性隔離掉。**

黏性 (stickiness) 則是讓你的系統在「穩定」與「分散風險」之間，取得可控的平衡。

當你把它當成路由層（而不是一堆零散設定），你就會開始自然地做這些事：

- 分層：profiles tiering（成本/穩定性）
- 分流：暫時性 vs 非暫時性錯誤
- 告警：disabled 代表需要人介入
- 治理：多 agent 的憑證一致性與最小權限

到這裡，你就不是在「用 OpenClaw」，而是在「用工程方法管理你的 LLM 供應鏈」。

---

## 參考資料

[^1]: thepagent/claw-info, *同一 Provider 的 Auth Profiles 輪換（Rotation / Failover）*, https://raw.githubusercontent.com/thepagent/claw-info/main/docs/profile_rotation.md （擷取時間：2026-02-24）

---

## 授權

本文（本次整理與增補內容）採用 **CC BY-NC-SA 4.0**（姓名標示-非商業性-相同方式分享）。引用之原始內容各自保留其原授權/來源標示。
