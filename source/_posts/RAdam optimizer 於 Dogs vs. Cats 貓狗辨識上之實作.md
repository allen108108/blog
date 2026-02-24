---
title: "RAdam optimizer 於 Dogs vs. Cats 貓狗辨識上之實作"
date: 2019-10-08 01:18:22
categories:
- 實作 Implementation
image: https://i.imgur.com/T0wRLLh.png
mathjax: true
---


就在 Lookahead Optimizer 發表後的一個月，University of Illinois, Urbana-Champaign 的博士生 Liyuan Liu 發表了一個新的優化器 Rectified Adam (RAdam)，有別於 Lookahead ，RAdam 在某種程度上直接對 Adam 進行改良，使其兼具 Adam 及 SGD 的優點，改善 Adam 落入 Local Minimum 導致效果不夠好以及 SGD 收斂速度慢的缺點。

<!-- more -->

Learning Rate Warmup in Adam
---

Adam 是一種自適應學習率的演算法，而 Warm Up 是調整學習率的一種常見策略。在 RAdam 的論文 " *On the Variance of the Adaptive Learning Rate and Beyond* " 中，作者也對於 Warm Up 有一些討論。

所謂的 Warm up 就是在訓練初期，先利用較小(、遞增)的學習率來先讓整個模型有一定程度的收斂後，再讓學習率遞減來進行訓練。

在一個原始的 Adam 中進行訓練，Training Perplexity[^註1] 大概收斂在 500 左右，但若先進行 Warm up ，Training Perplexity 便可迅速降至 10 左右。



![](https://i.imgur.com/YJD9QOZ.png =400x)





下圖是一個在不同迭代次數時梯度的機率分布圖，這是一個三維視覺圖像，X軸指的是梯度大小，Y軸是不同的迭代次數，Z軸是各梯度大小的頻率。

上面的兩張圖顯示出，Adam 在沒有 Warm up 的情況下，初期的迭代梯度會有很劇烈的變化 ( 紅框標示處 )



![](https://i.imgur.com/j3pxCuV.jpg =500x)



這樣劇烈的變動表示，當我們資料不足，迭代次數不夠多，Adam 如果處在較差的 Local Minimum 附近，就會容易陷進去，而得不到更好的解，而整個訓練也會因為初始值選的不夠好而導致優化時間拉長、優化效率可能更差。

總歸來說，問題還是在於如果訓練初期樣本數缺乏就會導致 Adam 的收斂出現問題。

這個根本的問題在論文中也嘗試利用 Variance Reduction 解決，但是效能仍然相對於 Adam-warmup、Adam-2K 較差。因此這才延伸出 Rectified Adam 這個嶄新的優化器。

Rectified Adam (RAdam)
---

雖然 Warm up 有類似 Variance Reduction 的效果，也能獲得不差的優化結果，但是 Warm up 所需要的時間隨著資料集不同而有所差異，我們並無法確認 Warm up 的時間。因此作者利用一個整流器來製造出動態 Variance Reduction 的效果。

完整 RAdam 演算法如下 : 

![](https://i.imgur.com/kqLr2eL.png)

在論文裡面，RAdam 在 CIFAR-10 或是 ImageNet 上都有相對好的表現，除此之外，我認為有一個很有趣的現象值得討論

![](https://i.imgur.com/T0wRLLh.png)

上圖顯示了三種不同的優化器在不同的學習率上的表現，以往我們都會認為學習率作為一個超參數在決定模型的表現上也佔有一定的決定性，但從 RAdam 的數據看來，學習率從 0.003 到 0.1 這麼大的範圍內，RAdam 都能有穩定的優化效能，這是與其他的優化器不同的地方。

RAdam 的 robustness (強健性) 不只展現在學習率上，在各種資料上也幾乎都適用，有著不錯的優化效能。


Adam VS. RAdam
---

我在貓狗辨識上使用不含分類器的 VGG19 model，接上我們自己的一個簡單分類器，且所有權重均重新訓練。

我們將不同優化器的訓練結果儲存，以利後續的比較。

```python=
results={}
OPTIMIZERS=[Adam(lr=1e-3) , RectifiedAdam(lr=1e-3)]

for optim in OPTIMIZERS:
    base_model=VGG16(input_shape=(224,224,3),include_top=False)
    base_model.trainable = False
 
    model = models.Sequential()
    model.add(base_model)
    model.add(layers.Flatten())
    model.add(layers.Dense(256, activation='relu'))
    model.add(layers.Dense(1, activation='sigmoid'))
    model.summary()

    model.compile(optimizer=optim ,
                  loss='binary_crossentropy' , 
                  metrics=['accuracy'])
    model.fit_generator(train_batches,
                    steps_per_epoch = train_batches.samples / 64,
                    validation_data = valid_batches,
                    validation_steps = valid_batches.samples /64,
                    epochs = 100,
                    callbacks=[LR_function])
    
    train_acc=model.history.history['acc']
    train_loss=model.history.history['loss']
    val_acc=model.history.history['val_acc']
    val_loss=model.history.history['val_acc']
    
    exp_tag_name=f'exp-optimizer-{optim}'
    results[exp_tag_name] = {'train-acc':train_acc,
                          'train-loss':train_loss,
                          'val-acc':val_acc,
                          'val-loss':val_loss}    
```

訓練過後，我們將 Loss 與 Accuracy 視覺化

![](https://i.imgur.com/55DwJRn.png)
![](https://i.imgur.com/BFhsFfx.png)
(實線為 train-loss (accuracy)，虛線為 valid-loss(accuracy))

從我們的實驗中可以發現 RAdam 收斂速度快，且最後結果也均較 Adam 來的好。


後記
---

我自己使用的顯示卡是 GTX 1060 6g ，在使用 RAdam 的時候，有時候會碰到 OOM error，這樣的情況看似是顯示卡記憶體不敷使用所導致。最後的解決方式就是將 batch_size 降低使其可正常運行。

原本以為 Lookahead 同時操作兩組參數比較容易跑不動，沒想到 RAdam 的演算反而對算力的要求較高。

註釋
---

[^註1]:
Training Perplexity 可以視為一種衡量 model 的指標，Training Perplexity 越低就表示此 model 可以更好的預測樣本。