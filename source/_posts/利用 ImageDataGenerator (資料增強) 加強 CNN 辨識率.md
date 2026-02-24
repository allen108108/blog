---
title: 利用 ImageDataGenerator (資料增強) 加強 CNN 辨識率
date: 2019-10-07 04:12:36
categories:
- 實作 Implementation
image: https://i.imgur.com/PTZrgL1.png
mathjax: true
---


此篇文章主要著重在 ImageDataGenerator 的使用，以及是否能增進 CNN 的辨識率，因此文章內主要的 CNN Architecture 均沿用 " [*卷積神經網路 Convolutional Neural Network ( CNN ) 與 全連接神經網路 Fully Connected Feedforward Network 於 MNIST 上之實作*](http://bit.ly/2oZH5f6) " 一文。


<!-- more -->

## 程式碼及注意事項

```python=
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from sklearn.model_selection import train_test_split

from keras.datasets import mnist
from keras.preprocessing.image import ImageDataGenerator
from keras.callbacks import ReduceLROnPlateau

from keras.utils import to_categorical
from keras import layers
from keras import models
from keras import optimizers
```

`ImageDataGenerator` : 利用現有的資料經過旋轉、翻轉、縮放...等方式增加更多的訓練資料。

`ReduceLROnPlateau` : 當我們使用 gradient descent 進行權重更新時，如果 Learning rate 固定，很容易到後面 Loss 會降不下來，而這個套件便是可以在我們設定的 epoch 內，若 Loss 沒有下降，可以自動調整 Learning rate 衰減 (這邊我們會設定為 0.5 倍)

```python=
(train_data,train_label),(test_data,test_label)=mnist.load_data()

print(f'The Train Data shape : {train_data.shape}')
print(f'The Test Data shape : {test_data.shape}')
```

![](https://i.imgur.com/2jsrLGt.png)

```python=
train_data = train_data.reshape(-1,28,28,1).astype('float32')/255
test_data = test_data.reshape(-1,28,28,1).astype('float32')/255

train_label = to_categorical(train_label)
test_label = to_categorical(test_label)

train_data,val_data,train_label,val_label=train_test_split(train_data,train_label,test_size=0.25,random_state=13)
```

接下來就是 `ImageDataGenerator` 與 `ReduceLROnPlateau` 設定的重點

```python=
train_datagen=ImageDataGenerator(rotation_range=15 , 
                             width_shift_range=0.2 , 
                             height_shift_range=0.2 ,
                             shear_range=0.2 ,
                             zoom_range=0.2 , 
                             data_format='channels_last')


train_datagen.fit(train_data)

LR_function=ReduceLROnPlateau(monitor='val_acc',
                             patience=3,
                             # 3 epochs 內acc沒下降就要調整LR
                             verbose=1,
                             factor=0.5,
                             # LR降為0.5
                             min_lr=0.00001
                             # 最小 LR 到0.00001就不再下降
                             )
```

這邊值得注意的是 `ImageDataGenerator` 內的參數必須要依照我們的資料進行調整，舉例來說，如果我們今天是做一般貓狗照片的辨識，我們可以將 `rotation_rang` 上調 ( 貓狗照片不會因為大角度旋轉而改變 Label，但數字若旋轉角度太大會有可能變成另外一個數字 ) 、加上 `horizontal_flip = True` 或是 `vertical_flip = True` 這兩個參數...等等[^註1]。



但我們今天是做手寫數字的辨識，因此在參數上面必須多所斟酌，不然很有可能會訓練出一套極差的 model。

```python=
#進行 CNN 模型的建構
model=models.Sequential()

model.add(layers.Conv2D(32,(3,3),activation='relu',input_shape=(28,28,1)))
model.add(layers.MaxPooling2D((2,2)))
model.add(layers.Conv2D(64,(3,3),activation='relu'))
model.add(layers.MaxPooling2D((2,2)))
model.add(layers.Dropout(0.5))
model.add(layers.Conv2D(64,(3,3),activation='relu'))

model.add(layers.Flatten())
model.add(layers.Dense(64,activation='relu'))
model.add(layers.Dense(10,activation='softmax'))

model.compile(optimizer='Adam' , loss='categorical_crossentropy' , metrics=['accuracy'])
```

```python=
history = model.fit_generator(train_datagen.flow(train_data,train_label,batch_size=64), 
                              steps_per_epoch=train_data.shape[0]/64 , epochs=100,validation_data=(val_data,val_label),validation_steps=val_data.shape[0]/64,callbacks=[LR_function])
```
這邊也有許多要注意的地方 : 

* 由於我們在訓練資料上用了 `ImageDataGenerator`，因此在訓練的步驟我們必須要使用 `fit_generator`  (如果我們在訓練資料上使用任何的 generator 都應該這樣使用 )。

* 我們以往對於資料 `fit` 以後還要再做 `transform` ，但在 `ImageDataGenerator` 我們在 `fit` 完後，要把資料做轉換的步驟則是用 `.flow(X,y)` 或是 `.flow_from_directory(directory)`

* 一個 epoch 要跑完全部的訓練資料，所以若訓練資料有 n 筆，batch size 為 m ，那麼每一個 epoch 應該要跑 n/m 次，亦即 `steps_per_epoch`*`batch_size`= `len(training data)`，validation data 亦然。


```python=
loss,acc=model.evaluate(test_data,test_label)
print(f'The Accuracy on Test Data = {acc}')
```
![](https://i.imgur.com/pFdtSVH.png)

準確率可以到達將近 99.5%

```python=
acc = history.history['acc']
loss = history.history['loss']
val_acc=history.history['val_acc']
val_loss=history.history['val_loss']

epochs=range(100)

plt.plot(epochs,acc,'r-',label='Training Accuracy')
plt.plot(epochs,val_acc,'b-',label='Validation Accuracy')
plt.title('Training and Validation Accuracy')
plt.legend()

plt.show()
```

![](https://i.imgur.com/PTZrgL1.png)


```python=
plt.plot(epochs,loss,'r-',label='Training Loss')
plt.plot(epochs,val_loss,'b-',label='Validation Loss')
plt.title('Training and Validation Loss')
plt.legend()

plt.show()
```

![](https://i.imgur.com/9xSdNSC.png)

這邊蠻有趣的地方是，以往訓練資料的準確度應該要比驗證資料來的高，而 Loss 應該要比驗證資料來的低。但經過 `ImageDataGenerator` 後訓練資料在模型上的表現卻總是不及驗證資料。但最後測試資料的分數仍然有明顯的提高。


## 後記

其實這裡有一個問題，因為我們放了兩個變因進去 : `ImageDataGenerator` 與 `ReduceLROnPlateau`，所以是否真的是 `ImageDataGenerator` 導致準確度提高的呢 ?

為了處理這個問題，我也有另外做一個只有 `ReduceLROnPlateau` 的 model，準確度的確也有提高，但大約在 99.4% 左右，也就是說 `ImageDataGenerator` 的影響雖然存在，但是卻不高，或許在參數上面調整一下可能會可以有明顯的差距。抑或者，在數字辨識上的資料增強本來就不會有太大的效果 ?


註釋
---
[^註1]: 
詳細的參數設置可以參閱 [Keras Document](https://keras.io/preprocessing/image/)。