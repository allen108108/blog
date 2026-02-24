---
title: "零基礎實作：用 OpenClaw × AWS AgentCore Browser 打造 AI 自動化瀏覽器代理（終極版）"
date: 2026-02-24 13:07:27
categories:
- 實作 Implementation
- 技術教學
tags:
- OpenClaw
- AWS
- AgentCore
- Browser
- 自動化
- AI
---

> 本文為多篇技術實作紀錄的完整整合版，涵蓋從理論基礎、環境建置、核心腳本拆解、身分洗白 (Identity Sanitization)、Profile 持久化，到 OpenClaw SKILL 整合的完整工程生命週期。字數超過 5,000 字，適合從零開始的開發者。

---

## 目錄 (TOC)

- [導言：一場對抗雲端封鎖的工程旅程](#導言一場對抗雲端封鎖的工程旅程)
- [Part 1：概念理解（Why & What）](#part-1概念理解why--what)
  - [1.1 從痛點出發](#11-從痛點出發)
  - [1.2 核心名詞解釋表](#12-核心名詞解釋表)
  - [1.3 整體架構圖](#13-整體架構圖)
  - [1.4 控制面 vs. 運行時的本質區分](#14-控制面-vs-運行時的本質區分)
- [Part 2：環境建置（Step by Step）](#part-2環境建置step-by-step)
  - [2.1 AWS 帳號與 IAM 設定](#21-aws-帳號與-iam-設定)
  - [2.2 安裝必要套件](#22-安裝必要套件)
  - [2.3 S3 Bucket 建立與職責分工](#23-s3-bucket-建立與職責分工)
  - [2.4 環境變數配置（.env 完整清單）](#24-環境變數配置env-完整清單)
  - [2.5 OpenClaw 設定](#25-openclaw-設定)
- [Part 3：核心腳本逐行拆解（How It Works）](#part-3核心腳本逐行拆解how-it-works)
  - [3.1 核心連線腳本：aws-browser-link.py](#31-核心連線腳本aws-browser-linkpy)
  - [3.2 持久化記憶：aws-save-profile.py](#32-持久化記憶aws-save-profilepy)
  - [3.3 資源清理：aws-manual-stop-v2.py](#33-資源清理aws-manual-stop-v2py)
  - [3.4 輔助腳本快速索引](#34-輔助腳本快速索引)
- [Part 4：進階攻堅 — 身分洗白與 Profile 固化](#part-4進階攻堅--身分洗白與-profile-固化)
  - [4.1 為什麼數據中心 IP 會被封鎖？](#41-為什麼數據中心-ip-會被封鎖)
  - [4.2 Custom Browser vs. 全託管 Browser 的本質差異](#42-custom-browser-vs-全託管-browser-的本質差異)
  - [4.3 身分洗白的完整流程](#43-身分洗白的完整流程)
  - [4.4 aws-wash-and-save.py 腳本說明](#44-aws-wash-and-savepy-腳本說明)
  - [4.5 分段擷取協議（防逾時設計）](#45-分段擷取協議防逾時設計)
- [Part 5：OpenClaw SKILL 整合（把腳本串起來）](#part-5-openclaw-skill-整合把腳本串起來)
- [Part 6：進階維運（Advanced Ops）](#part-6進階維運advanced-ops)
  - [6.1 安全性與金鑰層級架構](#61-安全性與金鑰層級架構)
  - [6.2 成本控制：Session 生命週期管理](#62-成本控制session-生命週期管理)
  - [6.3 Debug 工具鏈](#63-debug-工具鏈)
  - [6.4 已知風險與應對](#64-已知風險與應對)
- [後記與建議（Outro）](#後記與建議outro)
- [參考資料](#參考資料)

---

## 導言：一場對抗雲端封鎖的工程旅程

在 AI 代理 (AI Agents) 的開發過程中，我們常會遇到一個極其棘手的瓶頸：**如何讓 AI 真正「看見」並「操作」複雜的 Web 介面？**

傳統的網頁爬蟲技術雖然成熟，但面對需要登入狀態、動態渲染，甚至包含複雜反爬蟲機制的網站時，開發者往往需要耗費大量時間維護主機環境、管理 IP 輪換，以及處理 Cookie 的保鮮問題。

筆者在開發自動化偵察工具時，也曾深陷無止盡的環境配置地獄。而當我試著將瀏覽器代理遷移到雲端，採用 **AWS AgentCore Browser** 之後，又踩進了另一個更深的坑：**主流社群平台對雲端數據中心 IP 的硬封鎖 (Hard Block)**。

直接進入 `/login` 頁面？封。注入 `auth_token` 然後跳轉首頁？也封。X.com（前身為 Twitter）的機器人偵測系統甚至能在瀏覽器完成第一次 WebSocket 握手前，就決定這次連線的命運。

這篇文章記錄的，不只是一個簡單的「環境建置教學」，而是一段從**被封鎖 → 逆向分析 → 身分固化 → 突圍成功**的完整工程攻堅旅程。如果你也在嘗試打造具備持久登入狀態的雲端 AI 自動化系統，這份筆記將為你省下至少三天的試錯成本。

---

## Part 1：概念理解（Why & What）

### 1.1 從痛點出發

傳統的爬蟲開發就像經營一家**個人手工作坊**。你需要自己找零件（主機）、管電力（IP 輪換）、還要確保工作坊的記憶不消失（Cookie 儲存）。工作坊關門（程式結束）後，所有狀態灰飛煙滅，下次開業又得從頭來過。

**AWS AgentCore Browser** 則提供了一種**工廠外包**的模式：

- 你不需要關心瀏覽器跑在哪台伺服器上。
- 你只需要發出一條 API 指令，AWS 就會為你啟動一個現成的 Chrome 實體。
- 任務結束後，AWS 自動回收資源，並可選擇性地將「記憶」（Profile）儲存起來，供下次使用。

但工廠也有代工廠的問題：**AWS 的 IP 是公開的企業用 IP 段**，很容易被目標網站的安全系統識別為「機器人工廠」，從而觸發更嚴格的封鎖邏輯。這就是我們後面需要「身分洗白」的根本原因。

---

### 1.2 核心名詞解釋表

| 名詞 | 白話說明 |
| :--- | :--- |
| **Browser** | AWS 雲端托管的 Chrome 瀏覽器實體，是所有操作的「硬體載體」。 |
| **Custom Browser** | 由用戶建立的自訂瀏覽器實體，具備更真實的硬體指紋，對比全託管版本更難被識別。 |
| **Session** | 一次「開機到關機」的使用週期。Session 逾時會自動釋放資源並計費結束。 |
| **Profile** | 儲存瀏覽器「記憶」的快照，包含 Cookie、LocalStorage、登入狀態，後端存儲在 S3 或 AWS 內部。 |
| **SigV4** | AWS 簽章協議 (Signature Version 4)，是所有 AWS API 請求必須通過的安全驗證機制。 |
| **automationStream** | AI 控制瀏覽器的 WebSocket 端點，是「AI 大腦」與「遠端 Chrome」之間的高速公路。 |
| **bedrock-agentcore-control** | 負責「資產管理」的 AWS API，如建立/刪除瀏覽器實體或 Profile。 |
| **bedrock-agentcore** | 負責「運行時操作」的 AWS API，如啟動 Session、儲存 Profile 狀態或強制停止。 |
| **身分洗白 (Identity Sanitization)** | 透過人類手動登入動作，讓 AWS 雲端節點獲得目標平台的信任，並將此信任快照固化至 Profile。 |

---

### 1.3 整體架構圖

這套系統的運作邏輯可以分為三個層次：**調度層（OpenClaw）、API 橋接層（Python Scripts）、執行層（AWS Cloud Chrome）**。

```
┌─────────────────────────────────────────────────────────────┐
│                   OpenClaw Agent (調度層)                    │
│                                                             │
│   SKILL.md ──────────────────────────► 任務決策中樞          │
│       │                                                     │
│       ├── [啟動] aws-browser-link.py                        │
│       │       │ start_browser_session()                     │
│       │       │ SigV4 預簽章                                 │
│       │       ▼                                             │
│       │   WSS URL (帶簽章的 WebSocket 地址)                  │
│       │       │                                             │
│       ├── [控制] browser tool (Playwright WebSocket)        │
│       │       │ 掛載至 AWS 遠端 Chrome                       │
│       │       │ snapshot / act / navigate / evaluate        │
│       │       ▼                                             │
│       │   ┌─────────────────────────────────┐              │
│       │   │     AWS AgentCore Browser        │              │
│       │   │   (真實 Chrome 執行環境)           │              │
│       │   │   Region: us-east-1 / ap-ne-1    │              │
│       │   └─────────────────────────────────┘              │
│       │                                                     │
│       ├── [儲存] aws-save-profile.py                        │
│       │       │ save_browser_session_profile()              │
│       │       │ 將 Session 狀態快照 → Profile (S3/AWS內部)   │
│       │       ▼                                             │
│       │   Profile ID (永久化的登入記憶)                       │
│       │                                                     │
│       └── [清理] aws-manual-stop-v2.py                      │
│               │ stop_browser_session()                      │
│               ▼                                             │
│           Session 銷毀（避免持續計費）                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
          │                          │
          ▼                          ▼
  bedrock-agentcore              bedrock-agentcore-control
  (運行時 API)                    (資產管理 API)
  - start/stop/save session       - create/delete browser
                                  - create/manage profile
```

---

### 1.4 控制面 vs. 運行時的本質區分

這是最容易混淆的概念，直接影響你使用哪個 API Endpoint：

| 面向 | AWS Service Name | 職責 |
| :--- | :--- | :--- |
| **控制面 (Control Plane)** | `bedrock-agentcore-control` | 資產的生命週期管理：建立、設定、刪除 Browser 實體與 Profile。頻率低，通常一次性操作。 |
| **運行時 (Runtime)** | `bedrock-agentcore` | 即時操作：啟動 Session、儲存 Profile 快照、強制終止 Session。頻率高，每次任務都會呼叫。 |

**簡單記憶法**：「control = 製造工廠的設備管理員，runtime = 現場工人」。

---

## Part 2：環境建置（Step by Step）

### 2.1 AWS 帳號與 IAM 設定

首先，你需要一組具備相關權限的 IAM 使用者 (IAM User) 金鑰。

請在 AWS IAM Console 建立一個新 User，並附加以下權限策略 (Policy)：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock-agentcore:*",
        "bedrock-agentcore-control:*",
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket",
        "s3:CreateBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

> ⚠️ **安全建議**：生產環境請遵循最小權限原則 (Principle of Least Privilege)，將 `s3:*` 範圍縮小至指定的 Bucket ARN，並移除不必要的 `bedrock-agentcore-control:DeleteBrowser` 等危險操作。

取得 Access Key ID 與 Secret Access Key 後，進入下一步。

---

### 2.2 安裝必要套件

核心依賴為 `boto3`（AWS Python SDK）。如果你的執行環境（如 OpenClaw 沙盒）限制了全域 pip 安裝，可使用 `-t` 參數將套件裝至指定目錄。

```bash
# 安裝至用戶目錄，避免全域衝突
pip install boto3 -t /home/node/bin

# 驗證安裝
python3 -c "import boto3; print(boto3.__version__)"
```

若你的腳本需要讀取 `-t` 安裝的套件，請在腳本頂部加入：

```python
import sys
sys.path.insert(0, '/home/node/bin')
import boto3
```

---

### 2.3 S3 Bucket 建立與職責分工

許多開發者會誤以為 Profile 數據直接儲存在他們建立的 S3 Bucket 裡，**這是一個常見的誤解**。

實際上，S3 Bucket 的用途取決於你使用的 Browser 類型：

| Browser 類型 | Profile 存儲位置 | 你的 S3 Bucket 用途 |
| :--- | :--- | :--- |
| **全託管 Browser (aws.browser.v1)** | AWS 加密內部儲存，你看不到也摸不到 | 預留用途（錄影、擴展組件） |
| **Custom Browser** | 你指定的 S3 路徑，完全透明可控 | 直接儲存 Profile 快照數據 |

這就是「Custom Browser」的核心價值：**數據主權**。你能完整掌控 Profile 的備份與版本管理。

建立 Bucket 指令：

```bash
aws s3 mb s3://openclaw-browser-profiles --region us-east-1
```

---

### 2.4 環境變數配置（.env 完整清單）

以下是整套系統所需的完整環境變數清單，建議在 OpenClaw 的設定或伺服器的 `.bashrc` 中統一管理：

```bash
# AWS 身份認證
export AWS_ACCESS_KEY_ID="你的_ACCESS_KEY"
export AWS_SECRET_ACCESS_KEY="你的_SECRET_KEY"
export AWS_REGION="us-east-1"           # 建議首選，us-east-1 服務最穩定

# AgentCore Browser 實體識別
# 使用全託管版本時填入 "aws.browser.v1"
# 使用 Custom Browser 時填入你的 Custom Browser ID
export AGENTCORE_BROWSER_ID="openclaw_custom_browser-kfNkq5ny9R"

# 持久化身分 Profile
export AGENTCORE_PROFILE_ID="XMAN_Allen_v1_Final-n6ln2KcAK7"

# Session 逾時設定（秒），建議不超過 3600（1 小時）
export AGENTCORE_SESSION_TIMEOUT=3600

# S3 Bucket（用於錄影或 Custom Browser Profile 存儲）
export AGENTCORE_S3_BUCKET="openclaw-browser-profiles"
```

---

### 2.5 OpenClaw 設定

在 `openclaw.json` 中確保以下設定正確：

1. **Browser Tool 已啟用**：後續我們會將 SigV4 簽章後的 WSS URL 直接注入給 OpenClaw 的 browser 模組。
2. **環境變數已繫結**：確保上述 `export` 的變數在 OpenClaw 的執行上下文中可被 Python 腳本讀取。
3. **腳本路徑可存取**：建議將所有 Python 腳本統一放置在 `/home/node/.openclaw/custom-skills/xman-recon/scripts/`。

---

## Part 3：核心腳本逐行拆解（How It Works）

### 3.1 核心連線腳本：aws-browser-link.py

這是整個專案最關鍵的組件，負責完成以下任務鏈：

```
讀取環境變數 → 啟動 AWS Session → 生成 SigV4 預簽章 WSS URL → 輸出給 OpenClaw 掛載
```

#### 段落 A：SigV4 手動簽章原理

為什麼需要「手動」簽章？因為 `boto3` SDK 能呼叫 `start_browser_session()` 取得 `automationStream` 的 WebSocket 端點，但**這個端點本身就是一個需要 SigV4 驗證的受保護資源**。`boto3` 只能幫你簽署 REST/HTTP 請求，無法直接為 WebSocket 長連接 URL 生成預簽章 (Presigned URL)。

我們需要模擬 AWS 官方的 V4 簽署流程，其數學本質是一個四層 HMAC-SHA256 金鑰派生鏈：

$$\text{kSigning} = \text{HMAC}(\text{HMAC}(\text{HMAC}(\text{HMAC}(\text{"AWS4" + SecretKey}, \text{Date}), \text{Region}), \text{Service}), \text{"aws4\_request"})$$

用現實中的類比理解：這就像是在信封上蓋了四層巢狀的防偽蠟封，每一層都由前一層的印章壓出，任何一層偽造都會導致整個簽章無效。

```python
import hmac, hashlib, datetime, os
from urllib.parse import urlencode, quote

def sign(key, msg):
    return hmac.new(key, msg.encode('utf-8'), hashlib.sha256).digest()

def get_signature_key(secret_key, date_stamp, region, service):
    """四層 HMAC-SHA256 金鑰派生"""
    k_date    = sign(("AWS4" + secret_key).encode("utf-8"), date_stamp)
    k_region  = sign(k_date, region)
    k_service = sign(k_region, service)
    k_signing = sign(k_service, "aws4_request")
    return k_signing

def create_presigned_ws_url(endpoint_url, access_key, secret_key, region,
                             service='bedrock-agentcore', session_token=None):
    """
    生成帶有 SigV4 簽章的 WebSocket Presigned URL
    WebSocket 連線使用 UNSIGNED-PAYLOAD 模式
    """
    now = datetime.datetime.utcnow()
    amzdate    = now.strftime('%Y%m%dT%H%M%SZ')
    datestamp  = now.strftime('%Y%m%d')

    # 構建標準查詢參數
    query_params = {
        'X-Amz-Algorithm':     'AWS4-HMAC-SHA256',
        'X-Amz-Credential':    f'{access_key}/{datestamp}/{region}/{service}/aws4_request',
        'X-Amz-Date':          amzdate,
        'X-Amz-Expires':       '900',   # 15 分鐘有效期
        'X-Amz-SignedHeaders': 'host',
    }
    if session_token:
        query_params['X-Amz-Security-Token'] = session_token

    canonical_querystring = urlencode(sorted(query_params.items()), quote_via=quote)
    # ... (後續簽章步驟省略，完整實作請參考 GitHub 倉庫)

    return f"{endpoint_url}?{canonical_querystring}&X-Amz-Signature={signature}"
```

#### 段落 B：啟動 Session 並掛載 Profile

這裡最關鍵的參數是 `profileConfiguration`，它決定瀏覽器啟動時是否攜帶先前固化的「登入記憶」：

```python
import boto3, os, sys
sys.path.insert(0, '/home/node/bin')

def start_browser_session_with_profile():
    browser_id = os.environ.get('AGENTCORE_BROWSER_ID', 'aws.browser.v1')
    profile_id = os.environ.get('AGENTCORE_PROFILE_ID', '')
    timeout    = int(os.environ.get('AGENTCORE_SESSION_TIMEOUT', 3600))
    region     = os.environ.get('AWS_REGION', 'us-east-1')

    runtime = boto3.client('bedrock-agentcore', region_name=region)

    # 構建請求參數
    params = {
        'browserIdentifier':    browser_id,
        'name':                 'openclaw-auto-session',
        'sessionTimeoutSeconds': timeout,
    }

    # 若存在 Profile ID，則掛載既有登入狀態
    if profile_id:
        params['profileConfiguration'] = {'profileIdentifier': profile_id}
        print(f"[INFO] 掛載 Profile: {profile_id}")
    else:
        print("[WARN] 未指定 Profile，將以全新狀態啟動瀏覽器")

    response = runtime.start_browser_session(**params)

    session_id     = response['sessionId']
    ws_stream_url  = response['automationStream']['streamEndpoint']
    print(f"[OK] Session 啟動成功: {session_id}")
    print(f"[OK] 原始 WSS 端點: {ws_stream_url}")

    # 生成 SigV4 預簽章 URL
    access_key = os.environ['AWS_ACCESS_KEY_ID']
    secret_key = os.environ['AWS_SECRET_ACCESS_KEY']
    signed_url = create_presigned_ws_url(ws_stream_url, access_key, secret_key, region)

    print(f"\n[READY] 請將以下 URL 複製至 OpenClaw browser tool:")
    print(signed_url)
    return session_id, signed_url

if __name__ == '__main__':
    start_browser_session_with_profile()
```

#### 段落 C：Profile 名稱自動補完邏輯

在實際使用中，我們可能記住的是 Profile 的友善名稱（如 `XMAN_Allen_v1`），而非完整 ID（如 `XMAN_Allen_v1_Final-n6ln2KcAK7`）。腳本中實作了「名稱解析」機制：

```python
def resolve_profile_id(client, name_or_id):
    """若傳入名稱，自動查找對應的完整 Profile ID"""
    if name_or_id.count('-') > 2:  # 推測為完整 ID
        return name_or_id

    profiles = client.list_browser_profiles(browserIdentifier=os.environ['AGENTCORE_BROWSER_ID'])
    for p in profiles.get('browserProfileSummaries', []):
        if p['name'] == name_or_id:
            return p['profileIdentifier']
    raise ValueError(f"找不到名稱為 '{name_or_id}' 的 Profile")
```

---

### 3.2 持久化記憶：aws-save-profile.py

**這是使用 AWS AgentCore Browser 最容易忽略卻最重要的一個步驟。**

每次在 Session 中完成登入、操作互動後，這些狀態只存在於記憶體中。若直接 stop，所有操作都將消失。必須先呼叫 `save_browser_session_profile` 進行快照。

```python
def save_profile(session_id: str):
    """
    將當前 Session 的瀏覽器狀態快照至指定 Profile
    ⚠️ 時序關鍵：必須在 stop_browser_session() 之前呼叫
    """
    browser_id = os.environ['AGENTCORE_BROWSER_ID']
    profile_id = os.environ['AGENTCORE_PROFILE_ID']
    region     = os.environ.get('AWS_REGION', 'us-east-1')

    runtime = boto3.client('bedrock-agentcore', region_name=region)

    try:
        runtime.save_browser_session_profile(
            browserIdentifier=browser_id,
            sessionId=session_id,
            profileIdentifier=profile_id
        )
        print(f"[OK] Profile 儲存成功！狀態已固化至: {profile_id}")
        print("[INFO] 建議等待 60 秒後再啟動新 Session，確保 AWS 完成非同步寫入。")
    except Exception as e:
        print(f"[ERROR] Profile 儲存失敗: {e}")
        raise
```

> 🔑 **時序規範（黃金法則）**：永遠遵循 **`Save → Stop`** 的操作順序，不可顛倒。就像在關電腦前先按「儲存文件」，這是負責任的開發者的基本素養。

---

### 3.3 資源清理：aws-manual-stop-v2.py

AWS AgentCore Browser 按使用分鐘計費。任務完成後必須立即銷毀 Session，即使設定了 `sessionTimeoutSeconds`，也不應該依賴自動逾時來省費，因為那代表資源白白運行。

```python
def stop_session(session_id: str):
    """
    主動銷毀 Session，終止計費
    ⚠️ 必須在 save_profile() 之後呼叫
    """
    browser_id = os.environ['AGENTCORE_BROWSER_ID']
    region     = os.environ.get('AWS_REGION', 'us-east-1')

    runtime = boto3.client('bedrock-agentcore', region_name=region)

    try:
        runtime.stop_browser_session(
            browserIdentifier=browser_id,
            sessionId=session_id
        )
        print(f"[OK] Session {session_id} 已成功銷毀，計費終止。")
    except runtime.exceptions.ResourceNotFoundException:
        print(f"[WARN] Session {session_id} 已不存在（可能已自動逾時），無需手動停止。")
    except Exception as e:
        print(f"[ERROR] 停止 Session 失敗: {e}")
        raise
```

---

### 3.4 輔助腳本快速索引

| 腳本 | 用途 | 適用場景 |
| :--- | :--- | :--- |
| `aws-list-browsers.py` | 列出帳號下所有 Browser 實體 | 初始環境確認、Debug |
| `aws-list-profiles.py` | 列出所有 Profile 及其狀態 | 管理多帳號登入狀態 |
| `aws-list-sessions.py` | 列出所有活躍中的 Sessions | Debug 殭屍 Session、確認關閉 |
| `aws-check-session.py` | 查詢特定 Session 的即時狀態與串流端點 | 診斷「後台有連線但前台看不到」的資訊差問題 |
| `aws-wash-and-save.py` | 身分洗白 + Profile 儲存一條龍 | 首次建立可信 Profile 時的推薦做法 |

---

## Part 4：進階攻堅 — 身分洗白與 Profile 固化

> 本章節是從多次實戰失敗中萃取的核心知識，也是本文最獨特的價值所在。

### 4.1 為什麼數據中心 IP 會被封鎖？

現代主流平台（特別是 X.com、Cloudflare 保護的網站）的反機器人系統採用了**多維度信號融合**分析，包括：

1. **IP 信譽評分**：AWS、GCP、Azure 的 IP 段長期出現在爬蟲活動中，信譽分數極低。
2. **硬體指紋 (Hardware Fingerprinting)**：WebGL Renderer、Canvas 2D 紋理等特徵，全託管瀏覽器的硬體特徵是標準化的，容易被識別。
3. **行為模式分析**：完美的滑鼠軌跡、精準的點擊時序，在人類的眼裡是「效率」，在機器人探測器的眼裡是「機器人」。
4. **Session 歷史**：一個從未登入過、Cookie 全空白的 Browser Session，在統計上幾乎不可能是真實用戶。

**全託管 `aws.browser.v1`** 在以上四個維度中，有三個容易被識別。這就是我們需要升級到 **Custom Browser** 並執行身分洗白的根本原因。

---

### 4.2 Custom Browser vs. 全託管 Browser 的本質差異

| 比較維度 | 全託管 (aws.browser.v1) | Custom Browser |
| :--- | :--- | :--- |
| **建立方式** | 直接使用，無需設定 | 透過 `bedrock-agentcore-control` API 建立 |
| **硬體指紋** | 標準化，容易被識別 | 更接近真實硬體，具備獨特指紋 |
| **Profile 存儲** | AWS 內部加密儲存 | 用戶指定的 S3 路徑，數據透明可控 |
| **IP 範圍** | 共享 AWS 數據中心 IP 池 | 共享 AWS 數據中心 IP 池（相同） |
| **適合場景** | 一般網頁自動化、快速原型 | 對抗嚴格反機器人系統的長期穩定任務 |

建立 Custom Browser 的 API 呼叫（透過 `bedrock-agentcore-control` 客戶端）：

```python
control = boto3.client('bedrock-agentcore-control', region_name=region)

response = control.create_browser(
    name='openclaw-custom-browser',
    browserNetworkConfiguration={
        'networkMode': 'PUBLIC'
    }
    # 可選：加入錄影設定
    # 'videoCaptureConfig': {'status': 'ENABLED', 's3Location': {'bucket': 'your-bucket'}}
)
browser_id = response['browserIdentifier']
print(f"Custom Browser 建立成功: {browser_id}")
```

---

### 4.3 身分洗白的完整流程

身分洗白（Identity Sanitization）是一個**一次性但至關重要**的初始化儀式。流程如下：

```
第一步：建立空白 Profile
    ↓
第二步：啟動 Session（掛載空白 Profile）
    ↓
第三步：透過 AWS Console Live View 建立 WebRTC 視訊通道
    ↓
第四步：人類手動操作 — 完成目標網站的正常登入
    ↓
第五步：執行一次真實的人類互動（如：點讚、查看訂閱列表等）
    ↓
第六步：立即調用 save_browser_session_profile()
    ↓
第七步：等待 60 秒（非同步寫入延遲）
    ↓
第八步：執行 stop_browser_session()
    ↓
第九步：更新環境變數 AGENTCORE_PROFILE_ID
    ↓
完成：此 Profile 現在具備「已洗白」的身份，可供全自動偵察使用
```

> 💡 **關鍵洞察**：步驟五的「真實人類互動」至關重要。單純的登入動作有時不足以讓 AWS 節點「逃脫」目標平台的監視名單。一次真實的社交互動（點讚、滾動瀏覽等）能有效建立「正常用戶行為」的歷史記錄，顯著降低後續連線被封鎖的機率。

---

### 4.4 aws-wash-and-save.py 腳本說明

此腳本將「身分洗白」流程中的自動化部分打包成一條龍服務：

```python
def wash_and_save(target_url: str, profile_name: str):
    """
    身分洗白一條龍腳本
    1. 建立新 Profile（若不存在）
    2. 啟動新 Session 並掛載
    3. 導航至目標網站
    4. 等待人類手動登入（透過 Live View）
    5. 收到確認後，自動 save & stop
    """
    # ... 建立 Profile
    control = boto3.client('bedrock-agentcore-control', region_name=region)
    profile_response = control.create_browser_profile(
        browserIdentifier=os.environ['AGENTCORE_BROWSER_ID'],
        name=profile_name,
        # Custom Browser 需要指定 S3 路徑
        # profileStorageConfiguration={'s3Location': {'bucket': S3_BUCKET}}
    )
    new_profile_id = profile_response['profileIdentifier']
    print(f"[OK] 新 Profile 建立: {new_profile_id}")

    # ... 啟動 Session
    session_id, signed_url = start_browser_session_with_profile(profile_id=new_profile_id)

    print(f"\n⚠️  請開啟 AWS Console Live View，手動完成登入")
    print(f"   目標: {target_url}")
    input("登入完成後，按下 Enter 繼續...")

    # 自動 Save & Stop
    save_profile(session_id)
    import time; time.sleep(60)  # 等待非同步寫入
    stop_session(session_id)

    print(f"\n✅ 身分洗白完成！請更新環境變數：")
    print(f"   export AGENTCORE_PROFILE_ID={new_profile_id}")
```

---

### 4.5 分段擷取協議（防逾時設計）

對於需要大量滾動（如抓取 30+ 篇推文）的任務，單次 JavaScript 執行時間過長會導致 WebSocket 隧道超時（`Code 1006`）。建議採用「分段執行」策略：

```python
def scroll_and_collect(browser_tool, total_scrolls=30, batch_size=10):
    """
    分段滾動協議：每 batch_size 次滾動執行一次數據回傳
    防止 WebSocket 長時間佔用導致的連線中斷
    """
    collected_data = []
    for batch_start in range(0, total_scrolls, batch_size):
        batch_end = min(batch_start + batch_size, total_scrolls)

        for i in range(batch_start, batch_end):
            browser_tool.evaluate("window.scrollBy(0, 800)")
            time.sleep(1.5)  # 模擬人類滾動節奏

        # 每批次完成後立即萃取數據
        batch_data = browser_tool.evaluate("""
            Array.from(document.querySelectorAll('[data-testid="tweet"]'))
                 .map(t => t.innerText)
        """)
        collected_data.extend(batch_data)
        print(f"[INFO] 完成批次 {batch_start}-{batch_end}，累計: {len(collected_data)} 筆")

    return collected_data
```

---

## Part 5：OpenClaw SKILL 整合（把腳本串起來）

最後一塊拼圖：讓 OpenClaw 的 AI 大腦知道如何**自動**協調這些腳本。一個成熟的 `SKILL.md` 設計如下：

```
任務觸發條件：
  當用戶要求「瀏覽特定網頁」、「偵察 X」、「執行網頁自動化」時

標準執行流程：

Phase 1 - 環境就緒確認
  1. 確認 AGENTCORE_BROWSER_ID 與 AGENTCORE_PROFILE_ID 環境變數已設定
  2. 若未設定 Profile，詢問用戶是否需要全新 Session 或執行洗白流程

Phase 2 - Session 建立
  3. 執行 aws-browser-link.py，取得 SigV4 簽章後的 WSS URL
  4. 將 WSS URL 傳入 browser tool 作為連線目標

Phase 3 - 任務執行
  5. 使用 browser tool (navigate / snapshot / act / evaluate) 執行 AI 任務
  6. 若需要大量滾動，採用分段擷取協議（每 10 次滾動回傳一次數據）

Phase 4 - 持久化與清理（嚴格遵守時序）
  7. [先] 執行 aws-save-profile.py —— 固化 Session 狀態
  8. [等] 等待 60 秒（確保非同步寫入完成）
  9. [後] 執行 aws-manual-stop-v2.py —— 銷毀 Session，停止計費

異常處理：
  - 若 Session 啟動失敗：檢查 IAM 權限與 Region 設定
  - 若遭遇「Something went wrong」：提示用戶開啟 Live View 進行人工點擊 Retry
  - 若 WebSocket 斷線 (Code 1006)：檢查本地防火牆，改用分段擷取協議
```

---

## Part 6：進階維運（Advanced Ops）

### 6.1 安全性與金鑰層級架構

整套系統採用三層金鑰防禦架構：

```
層級 1（核心）: AWS IAM Access Keys
  ↓ 用於 SigV4 簽章，控制所有 AWS API 呼叫的授權
  ↓ 必須使用環境變數，嚴禁硬編碼至任何程式碼或版本控制系統

層級 2（身份）: AWS Browser Profile
  ↓ 封存了目標網站的登入狀態（Cookie/LocalStorage）
  ↓ 儲存在 AWS 加密環境中（全託管）或指定 S3（Custom Browser）
  ↓ 透過 AGENTCORE_PROFILE_ID 引用，無需直接傳遞敏感 Cookie

層級 3（備援）: 目標網站的原始憑證（如 X_COOKIE / auth_token）
  ↓ 當 Profile 意外失效時，用於執行 JS 暴力注入重建 Session
  ↓ 安全性最低，僅作緊急備援用途
```

**⚠️ 嚴禁事項**：
- 不得將 Access Keys 提交至 Git 倉庫
- 不得在 Notion 或任何公開筆記系統中儲存原始金鑰
- 建議定期輪換 IAM Access Keys（每 90 天）

---

### 6.2 成本控制：Session 生命週期管理

AWS AgentCore Browser 的計費邏輯是**「Session 活躍分鐘數」**，理解這點能幫你大幅降低成本。

**省錢原則：**

1. **絕不依賴逾時自動停止**：設定了 `sessionTimeoutSeconds=3600`，不代表你可以不手動 Stop。若任務 10 分鐘完成，剩下 50 分鐘都是純浪費。
2. **殭屍 Session 偵測**：定期執行 `aws-list-sessions.py` 確認無孤兒 Session 持續計費。
3. **單任務單 Session**：避免在一個長時間 Session 中執行多個不相關任務，難以追蹤成本歸因。

**快速偵測殭屍 Session：**

```bash
# 列出所有活躍 Session（若輸出為空則安全）
python3 aws-list-sessions.py | grep "ACTIVE\|RUNNING"
```

---

### 6.3 Debug 工具鏈

| 問題症狀 | 診斷工具 | 解決方向 |
| :--- | :--- | :--- |
| Session 啟動失敗 | `aws-list-browsers.py` | 確認 Browser ID 正確，檢查 IAM 權限 |
| WebSocket 連線失敗 (`Code 1006`) | 本地防火牆日誌 | 檢查 DCV 視訊串流是否被攔截 |
| Profile 儲存後 S3 為空 | 確認 Browser 類型 | 全託管 Browser 的 Profile 不走 S3，需改用 Custom Browser |
| 登入後立即被封鎖 | 手動開啟 Live View 觀察行為 | 執行身分洗白，加入人類互動行為 |
| Session 已 Stop 但繼續計費 | AWS Cost Explorer | 可能存在孤兒 Session，執行 `aws-list-sessions.py` |
| `list_browser_profiles` 回傳空陣列 | `aws-list-browsers.py` | 確認 AGENTCORE_BROWSER_ID 與 Profile 所屬 Browser 一致 |

---

### 6.4 已知風險與應對

**風險 1：IP 隨機攔截**

由於 AWS IP 段屬於數據中心，即使已完成身分洗白，偶爾仍可能遭遇「Something went wrong」。

**應對**：Profile 已固化，遭遇攔截時只需在 Live View 中點擊一次「Retry」即可解鎖，無需重新登入。

**風險 2：非同步寫入延遲**

`save_browser_session_profile()` API 回應成功不代表數據已完全寫入持久化儲存。

**應對**：在 Save 與 Stop 之間強制等待 60 秒（建議），生產環境可增至 120 秒。

**風險 3：Region 不一致**

若 Browser 建立在 `us-east-1`，Session 操作卻使用 `ap-northeast-1` 的 Endpoint，會觸發 404 或權限錯誤。

**應對**：所有操作統一使用 `AWS_REGION` 環境變數，確保 `boto3.client()` 的 `region_name` 全程一致。

---

## 後記與建議（Outro）

將瀏覽器代理托管至雲端，是 AI 自動化從「玩具」邁向「工程化」的重要一步。雖然增加了雲端成本與初始設定的複雜度，但其帶來的**高可用性、IP 安全性，以及狀態持久化**能力，是本地運行難以企及的。

回顧這段工程旅程，有幾個深刻的反思值得分享：

1. **不要假設雲端 = 匿名**。AWS IP 的公開性使得「躲在雲端」這個預設假設完全錯誤。身分洗白是必要的初始化儀式，而不是可選的優化項目。

2. **Profile 是最有價值的資產**。一個成功洗白的 Profile，比任何代碼技巧都更重要。保護它、備份它，就像保護你的私鑰一樣。

3. **時序是一切**。`Save → Stop` 的順序不是建議，是鐵律。任何一次的順序錯誤，都意味著小時級別的重工成本。

4. **分段執行不是妥協，是架構**。將長時間任務分拆成批次，不僅能避免超時，還能獲得更好的可觀測性與容錯能力。

隨著 AWS AgentCore 的持續進化，未來我們可以期待更豐富的 Profile 管理能力，以及更完善的 Custom Browser 硬體指紋配置選項。屆時，「真實 AI 代理的雲端滲透」將不再是科幻，而是工程標配。

---

## 參考資料

- [AWS Bedrock AgentCore Documentation](https://aws.amazon.com/bedrock/)
- [OpenClaw Browser Tool Specifications](https://docs.openclaw.ai)
- [AWS Signature Version 4 Signing Process](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)
- [Playwright WebSocket Remote Connection](https://playwright.dev/docs/api/class-browsertype#browser-type-connect)

---

### 聲明

本文章內容採用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 許可協議。歡迎在署名且非商業用途的前提下轉載與改作。

[^1]: **SigV4 技術補充**：AWS Signature Version 4 是極為嚴謹的安全協議，要求對請求的內容（包含時間戳、標頭、路徑、查詢參數）進行多層次雜湊處理，且有效期通常不超過 15 分鐘，確保即使簽章被截取也無法重放攻擊 (Replay Attack)。

[^2]: **非同步寫入說明**：AWS Profile 的物理寫入涉及加密、跨區域複製等後台流程，API 回應「成功」只代表請求已被接受並排入隊列，實際完成時間通常為 10-60 秒不等，高負載時可能更長。