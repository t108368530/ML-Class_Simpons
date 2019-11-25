Simpons 角色辨識
===

## 目錄

* [Simpons](#Simpons)
  * [說明](#說明)
  * [程式流程圖](#程式流程圖)
  * [結果分析](#結果分析)


## 說明
1. 將資料集切割成：
    * 70%(訓練用)
    * 30%(驗證用)
4. 利用Pre-Train Model(ResNet50)去做訓練


程式流程圖
---
#### 資料切割

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/split_train%26valid.png)

#### 訓練＆驗證＆預測

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/train.png)


結果分析
---
#### Train - 1
(epoch 20,size=(150,150))

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/tensorboard1.png)

`由圖可見使用預先訓練模型在剛開始前幾輪就有不錯的準確率了`

#### Train - 2
(epoch 30,size=(224,224))

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/tensorboard2.png)

#### 混淆矩陣

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/CM.png)




