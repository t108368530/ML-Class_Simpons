Simpons 角色辨識  
===

## 目錄  

* [Simpons](#Simpons)  
  * [說明](#說明)  
  * [程式流程圖](#程式流程圖)  
     * [資料切割](#資料切割)  
     * [訓練＆驗證＆預測](#訓練-AND-驗證-AND-預測)  
  * [程式](#程式)  
    * [資料切割](#訓練資料切割)  
    * [創建模型](#創建模型)  
    * [圖片處理](#圖片處理)  
    * [訓練模型](#訓練模型)  
    * [預測＆生成CSV](#預測-AND-生成CSV)  
  * [結果分析](#結果分析)  
     * [訓練與驗證 - 1](#訓練與驗證---1)  
     * [訓練與驗證 - 2](#訓練與驗證---2)  
     * [混淆矩陣](#混淆矩陣)  
  * [網路流程圖](#網路流程圖)  
     * [ResNet50-V2](#ResNet50-V2)  
  
  


 說明  
 ---
 
1. 將資料集切割成：  
    * **70%**(**訓練**用)  
    * **30%**(**驗證**用)  
4. 利用Pre-Trained Model(ResNet50)當作前級  




程式流程圖  
---
#### 資料切割  

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/split_train%26valid.png)  

#### 訓練 AND 驗證 AND 預測  

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/train.png)  


程式  
---

#### 訓練資料切割  
```python=
import os
from shutil import copyfile

def check(splitNum):
    for dirname in os.listdir("RAW_DATA"):
        for (dirpath, dirnames, filenames) in os.walk(
            os.path.join("RAW_DATA", dirname)
        ):
            yield dirname, int(len(filenames) * splitNum), filenames


def main():
    for dirname, splitnum, files in check(0.7):
        print(dirname, len(files))
        for file in files[:splitnum]:
            sorce = os.path.join("RAW_DATA", dirname, file)
            destination = os.path.join("train", dirname, file)
            copyfile(sorce, destination)
        for file in files[splitnum:]:
            sorce = os.path.join("RAW_DATA", dirname, file)
            destination = os.path.join("valid", dirname, file)
            copyfile(sorce, destination)
```

#### 創建模型
```python=
def create_model():
    ResNet = keras.applications.ResNet50V2(
        include_top=False, weights="imagenet", input_shape=(reSize[0], reSize[1], 3)
    )
    return models.Sequential(
        [
            ResNet,
            layers.GlobalAveragePooling2D(),
            layers.Dense(512, activation="relu"),
            layers.Dense(20, activation="softmax"),
        ]
    )
```

#### 圖片處理  
```python=
train = keras.preprocessing.image.ImageDataGenerator(
    rescale=1.0 / 255,
    zoom_range=0.1,
    width_shift_range=0.05,
    height_shift_range=0.05,
    horizontal_flip=True,
)
valid = keras.preprocessing.image.ImageDataGenerator(rescale=1.0 / 255)
test = keras.preprocessing.image.ImageDataGenerator(rescale=1.0 / 255)

trainGenerator = train.flow_from_directory(
    "train",
    target_size=reSize,
    batch_size=batchSize,
    class_mode="categorical",
    color_mode="rgb",
)

validGenerator = valid.flow_from_directory(
    "valid",
    target_size=reSize,
    batch_size=batchSize,
    class_mode="categorical",
    color_mode="rgb",
)

testGenerator = test.flow_from_directory(
    "test",
    target_size=reSize,
    batch_size=1,
    class_mode="categorical",
    shuffle=False,
    color_mode="rgb",
)
```
#### 訓練模型  
```python=
model = create_model()
model.compile(
    optimizer=keras.optimizers.SGD(lr=0.01, momentum=0.9, decay=1e-6, nesterov=True),
    loss="categorical_crossentropy",
    metrics=["accuracy"],
)
model.fit_generator(
    trainGenerator,
    steps_per_epoch=trainGenerator.samples // batchSize,
    epochs=30,
    validation_data=validGenerator,
    validation_steps=validGenerator.samples // batchSize,
    callbacks=[tensorboard_callback],
)
```


#### 預測 AND 生成CSV  
```python=
def submission():
    predict = model.predict_generator(testGenerator)
    predict = np.argmax(predict, axis=1)
    labels = trainGenerator.class_indices
    label = dict((v, k) for k, v in labels.items())
    predictions = [label[k] for k in predict]
    idList = []
    for filename in testGenerator.filenames:
        idList.append(int(filename[2:-4]))
    sub = pd.DataFrame(idList)
    sub.columns = ["id"]
    sub["character"] = predictions
    sub.sort_values(by=["id"]).to_csv("Submussion.csv", index=False)


submission()
```

結果分析  
---

#### 訓練與驗證 - 1  
(epoch 20,size=(150,150))

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/tensorboard1.png)  

`由圖可見使用預先訓練模型在剛開始前幾輪就有不錯的準確率了`  


#### 訓練與驗證 - 2  
(epoch 30,size=(224,224))  

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/tensorboard2.png)  

#### 混淆矩陣  

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/CM.png)  

#### 卷積第一層  

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/conv1.png)


網路流程圖  
---

#### ResNet50-V2  

![image](https://github.com/t108368530/ML-Class_Simpons/blob/master/png/ResNet50-V2-simpons.png)  

[`TOP`](#目錄)  
