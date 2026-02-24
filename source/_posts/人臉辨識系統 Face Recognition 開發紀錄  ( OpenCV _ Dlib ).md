---
title: "人臉辨識系統 Face Recognition 開發紀錄  ( OpenCV / Dlib )"
date: 2020-04-16 18:14:27
categories:
- 專案 Project
image: https://i.imgur.com/W11yLoi.png
mathjax: true
---


此專案利用 Pre-train 好的 Dlib model，進行人臉辨識　(Face Detection)　，並且實現僅用一張照片作為 database　就可以作出達到一定效果的人臉識別 (Face Recognition)。 除此之外，更加入了活體偵測 (Liveness Detection) 技術，以避免利用靜態圖片通過系統識別的問題。

<!-- more -->

![](https://i.imgur.com/W11yLoi.png)

整個 pipeline 是由多個模型所組成，其中包含 : 

* 臉部的偵測
* 人臉識別
* 雙眼的偵測
* 開閉眼識別
 
最後利用開閉眼的偵測來決定是否將人臉識別的結果顯示出來。

{%youtube CvZQnJfuP6Q %}

Dlib
---

Dlib 是一套基於 C++ 的機器學習工具包，藉由 Dlib 可以使用這些機器學習工具在任何的專案上，目前無論在機器人、嵌入設備、移動設備甚至是大型高效運算環境中都被廣泛使用。

### 在 Window 系統中安裝 Dlib

Dlib 跟許多開發套件相同，在 Windows 系統下非常容易出現安裝上的問題，因此在安裝上面需要花一些時間避坑，以下是筆者安裝成功的一些方法。

* **`pip` 安裝**

簡單一點的方法就是直接在命令提示字元中輸入 `pip` 進行安裝

```
$ pip install dlib
```
上述方式在環境不同的狀況下，可能因為一些依賴項或是系統問題，很有可能會安裝失敗，筆者改輸入下列命令，即可安裝成功，但這樣的安裝方式並不會安裝最新版本的 Dlib，而安裝的 Dlib 也無法調用 GPU。

```
$ python -m pip install https://files.pythonhosted.org/packages/0e/ce/f8a3cff33ac03a8219768f0694c5d703c8e037e6aba2e865f9bae22ed63c/dlib-19.8.1-cp36-cp36m-win_amd64.whl#sha256=794994fa2c54e7776659fddb148363a5556468a6d5d46be8dad311722d54bfcf
```

* **從 Sourse Code 進行安裝**

若是希望可以調用 GPU 運算或是要確保安裝最新版本的 Dlib ，最推薦的方式還是要直接從 Source Code 進行安裝。

首先，先將專案整個 clone 至本機資料夾中
```
$ git clone https://github.com/davisking/dlib
```

接著，請確認電腦中有安裝 Visual Studio 2015，根據實際上測試的結果，Visual Studio 2015 以外的版本都會導致編譯失敗。

最後在專案位址下輸入
```
$ python setup.py install
```

正常情況下，在 Windows 系統中利用此方法進行安裝，應該可以調用 GPU ，但筆者仍然無法成功使用 GPU，這部分的問題仍試圖釐清中。



人臉識別 Face Detection & Recognition
---

不管是用什麼方式進行人臉識別，基本上都要經過兩個階段的步驟

* 人臉對齊
* 身分判斷

先對一張照片、一張 Frame 找出人臉的位置進行對齊，再利用模型進行特徵萃取來比對出身分。在這個專案中，我們利用 Dlib 本身的臉部偵測器來進行人臉對齊後，再使用 Imperial College London 在 [ibug 300-W dataset](https://ibug.doc.ic.ac.uk/resources/facial-point-annotations/) 所訓練的[模型](https://github.com/AKSHAYUBHAT/TensorFace/blob/master/openface/models/dlib/shape_predictor_68_face_landmarks.dat)由鏡頭中人臉與資料庫中的人臉同時萃取出關鍵點後進行 Embedding，再進行身分比對。

### 流程簡述

1. 先收集欲辨識的人正面照片 ( 甚至可以收集多角度照片 )於特定資料夾 (此處設為 `./rec` )中，並將檔名設為人名。
2. 使用 `dlib.get_frontal_face_detector()` 擷取資料夾中照片的人臉，再利用 `dlib.shape_predictor()` 取出臉部 68 個關鍵點。
3. ` dlib.face_recognition_model_v1().compute_face_descriptor()` 將 68 個關鍵點進行嵌入成一個 128 維的向量 $\nu_1,\nu_2,\cdots$。
4. 相同的方式將鏡頭中的人臉也嵌入成 128 維的向量 $\nu$。
5. 計算 $\nu$ 與 $\nu_1,\nu_2,\cdots$ 個別計算歐式距離，最接近者及判定其身分。


### 模型路徑 Model Path

我們先將要使用的模型路徑設定好，

```python=
# 人臉對齊
detector = dlib.get_frontal_face_detector()

# 人臉關鍵點模型
predictor = dlib.shape_predictor( 'shape_predictor_68_face_landmarks.dat')

# 128維向量嵌入模型
face_rec_model_path = "dlib_face_recognition_resnet_model_v1.dat"
facerec = dlib.face_recognition_model_v1(face_rec_model_path)
```

### 人臉嵌入 Face Embedding

將每一個資料庫中的資料進行人臉各嵌入成一個 128 維度向量，並且收集所欲判定的人名。

```python=
# 比對人臉描述子列表
descriptors = []

# 欲比對人臉名稱列表
candidate = []

# 針對比對資料夾裡每張圖片做比對:
for f in glob.glob(os.path.join(faces_folder_path, "*.jpg")):
    base = os.path.basename(f)
# 依序取得圖片檔案人名
    candidate.append(os.path.splitext(base)[ 0])
    img = io.imread(f)
# 1.人臉偵測
    dets = detector(img, 0)
    for k, d in enumerate(dets):
# 2.特徵點偵測
        shape = predictor(img, d)
# 3.取得描述子，128維特徵向量
        face_descriptor = facerec.compute_face_descriptor(img, shape)    
# 轉換numpy array格式
        v = numpy.array(face_descriptor)
        descriptors.append(v)
```

相同的方式，配合鏡頭擷取出來的 Frame 來將畫面中的人臉嵌入成 128 維向量。

### 身分判別 Face Recognition

```python=
# 計算歐式距離
for i in descriptors:
    dist_ = np.linalg.norm(i -d_test)
    dist.append(dist_)
# 將比對人名和比對出來的歐式距離組成一個dict
cd = dict( zip(candidate,dist))
cd_sorted = sorted(c_d.items(), key = lambda d:d[1])
```

使用 `np.linalg.norm()` 直接計算嵌入向量的距離，並將人名及相對應計算出來的距離以 Dictionary 方式呈現，排序後取最高者作為判斷結果。

上面的流程就是單純人臉身分判斷的流程及部分程式碼解析，這樣的方式應用在 Windows 系統中 Inference time 稍長大概是唯一的問題，其判別的準確度筆者認為並不算太差。

![](https://i.imgur.com/mGUgrGI.jpg)


活人偵測 Liveness Detection
---

在筆者實作人臉辨識專案的同時，意外地看到了一篇挺有趣的新聞 : " [刷臉被破解！小學生拿「彩色照」秒解鎖　側臉、偷拍都能開](https://www.setn.com/News.aspx?NewsID=619777) "，讓筆者不禁思考，如何可以在不另外安裝硬體設備 (譬如 : 深度相機、紅外線鏡頭...等) 下，盡可能地避免這樣的狀況發生 ?

搜尋了一些資料，以目前的狀況大約有兩個方向可行 : (1) 直接訓練一個 Deep Learning Model 判斷人臉真假 ，(2) 利用眨眼的過程進行真實人臉的判斷。在此專案中，筆者採用了後者的方式進行 Liveness Detection。

眨眼偵測，筆者使用了 [Jordan Van Eetveldt](https://towardsdatascience.com/real-time-face-liveness-detection-with-python-keras-and-opencv-c35dc70dafd3) 預訓練好的左右眼開闔判別模型，先偵測鏡頭捕捉的眼睛是開啟還是閉眼。

再來配合簡單的邏輯判斷，當偵測眼睛的過程中出現了開、閉、開的過程，即判斷為真人，將前述的人臉辨識結果呈現出來。

### 定義函式

這邊主要定義三個函式，載入預訓練模型、進行開閉眼預測、以及邏輯判斷眨眼狀態 : 

```python=
def isBlinking(history, maxFrames):
    # 偵測眼睛狀態是否有開、閉、開這樣的連續動作出現
    for i in range(maxFrames):
        pattern = '1' + '0'*(i+1) + '1'
        if pattern in history:
            return True
    return False

def predict(img, model):
	img = Image.fromarray(img, 'RGB').convert('L')
	img = imresize(img, (24,24)).astype('float32')
	img /= 255
	img = img.reshape(1,24,24,1)
	prediction = model.predict(img)
	if prediction < 0.1:
		prediction = 'closed'
	elif prediction > 0.9:
		prediction = 'open'
	else:
		prediction = 'idk'
	return prediction

def load_model():
	json_file = open('model.json', 'r')
	loaded_model_json = json_file.read()
	json_file.close()
	loaded_model = model_from_json(loaded_model_json)
	# load weights into new model
	loaded_model.load_weights("model.h5")
	loaded_model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'])
	return loaded_model
```


### 載入模型 Load Model

```python=
open_eye_cascPath = 'C:\pythonwork\Face detection\haarcascade_eye_tree_eyeglasses.xml'
left_eye_cascPath = 'C:\pythonwork\Face detection\haarcascade_lefteye_2splits.xml'
right_eye_cascPath ='C:\pythonwork\Face detection\haarcascade_righteye_2splits.xml'

open_eyes_detector = cv2.CascadeClassifier(open_eye_cascPath)
left_eye_detector = cv2.CascadeClassifier(left_eye_cascPath)
right_eye_detector = cv2.CascadeClassifier(right_eye_cascPath)

model = load_model()
```

### 真人判定 Liveness Detection

一開始，系統僅會偵測眼睛，雖然同時也會對人臉進行辨識，卻不會顯示出來。而這個部分定義了，在什麼樣子的狀態下，才會將人臉辨識的結果呈現出來。

```python=
if isBlinking(eyes_detected[rec_name],3):
    cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)
    # Display name
    y = y - 15 if y - 15 > 15 else y + 15
    cv2.putText(frame, rec_name, (x, y),     
```



後記
---

其實看了此文介紹的人臉辨識，用了兩三個模型進行最終的人臉判別，其實如果有看過 FaceNet 論文 "[*FaceNet: A Unified Embedding for Face Recognition and Clustering*](https://arxiv.org/abs/1503.03832)" 其概念幾乎完全一樣，但是用了更快的方法來達成而已，或許，這個專案未來可以試著用 FaceNet 來替代 Dlib 的方式進行辨識。



另外，這種 Liveness Detection 的方式是否能完全杜絕人臉辨識被破解 ? 其實不然，基本上利用一個人臉有眨眼的影片或許也可以破解這個部分，但是正如筆者所提到的，真正萬無一失的辦法還是得另外安裝硬體設備，單純使用深度學習的方式或許盡可能可以做到的就是如此。

參考資料
---

1. [Real-time face liveness detection with Python, Keras and OpenCV](https://towardsdatascience.com/real-time-face-liveness-detection-with-python-keras-and-opencv-c35dc70dafd3)
2. [Liveness Detection with OpenCV](https://www.pyimagesearch.com/2019/03/11/liveness-detection-with-opencv/)
3. [基於python語言使用OpenCV搭配dlib實作人臉偵測與辨識](https://tpu.thinkpower.com.tw/tpu/articleDetails/950)

