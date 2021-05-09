# **使用 Jetson Nano 進行口罩辨識**  
## 需要準備的東西   
Jetson Nano 開發板  
SD 卡與讀卡機    
跨接器  
變壓器  
攝影機  
螢幕、鍵盤、滑鼠  
## **流程**  
從官方下載映象檔燒入 SD 卡中  
下載網址:https://developer.nvidia.com/jetpack-sdk-441-archive      
安裝完系統開機後先使用指令更新軟體  
`sudo apt-get update`  
`sudo apt-get upgrade`  
`sudo apt-get install nano -y`  
### **建構 darknet 環境**
在終端機輸入  
 `git clone https://github.com/AlexeyAB/darknet.git`  
`cd darknet`  
修改 darknet 資料夾中 Makefile 文件內容  
GPU=1  
CUDNN=1  
CUDNN_HALF=1  
OPENCV=1  
AVX=0  
OPENMP=0  
LIBSO=1  
NVCC=/usr/local/cuda/bin/nvcc  
修改完後儲存  
在 dreaknet 資料夾底下叫出控制台輸入  
`make`  
下載YOLOv4預設權重  
`wget https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v3_optimal/yolov4.weights`  
下載好後進行測試確認 darkent 有正確安裝  
`./darknet detector test ./cfg/coco.data ./cfg/yolov4.cfg ./yolov4.weights data/dog.jpg -i 0 -thresh 0.25`    
如果成功安裝測試結果如下  
 ![Image text](https://github.com/guizaida/How-to-do-jetson-nano-maskcheck/blob/31bb971b80d0a46909610c9327506a528ac685e5/img/111.jpg)  
## **建構口罩辨識模型**  
### 由於是在 Jetson Nano上 運行我們採用的模型是 YOLOv4-tiny.cfg 模型
下載模型權重這個已經訓練好 29 層  
`wget https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v4_pre/yolov4-tiny.conv.29` 
## 準備訓練資料  
下載圖片中有戴口罩的照片，用lablImg標註資料或者前往公開圖庫下載以標記好的照片  
如果是自行標註圖片在 **lablImg** 上記得選取 YOLO 模式這樣就不再需要進行標籤轉換  
![Image text](https://github.com/guizaida/How-to-do-jetson-nano-maskcheck/blob/31bb971b80d0a46909610c9327506a528ac685e5/img/112.jpg)    
在darkenk/data裡面創一個資料夾(自行命名)將剛剛標記好或者下載的圖片與標籤放進去  
複製一份圖片標籤檔放入darkenk/data/labels  
使用imgcut.py將訓練資料分割  
## **修改YOLOv4-tiny.cfg**  
打開改YOLOv4-tiny.cfg進行以下修改  
batch=16 原本是64用jetson nano 進行訓練的話會過載所以設小一點  
subdivisions=2  
max_batches=5000  
classes=2 有兩處都要修改    
filters=21  有兩處值為255的改成21其他不變  
## **建構 obj.data , obg.name 檔案**  
在 draknet/data 資料夾下創建 obj.data 跟 obj.name 兩個檔案分別輸入  
### **obj.name**  
複製圖片資料夾裡面 classes.txt 的內容  
### **obj.data**
classes = 2  
train = train.txt的路徑   
valid = test.txt的路徑  
names = obj.names的路徑  
backup = darknet/data/backup-tiny  
## **開始訓練**  
如有安裝風扇記得使用 jtop 打開  
在darknet資料夾下開啟終端機輸入  
`./darknet detector train data/obj.data cfg/yolov4-tiny.cfg ./yolov4-tiny.conv.29 -dont_show -map`  
## **驗證**
在訓練完成後要驗證模型的準確率  
準備一些口罩照片放入 data 資料夾裡面然後在 draknet 開啟終端機輸入  
`./darknet detector test data/obj.data cfg/yolov4-tiny.cfg  darknet/data/backup-tiny/yolov4-tiny_best.weights   測試圖片路徑 -i 0 -thresh 0.25`  
使用攝影機進行辨識  
`./darknet detector demo  cfg/yolov4-tiny.cfg  darknet/data/backup-tiny/yolov4-tiny_best.weights -c 0`
![Image text](https://github.com/guizaida/IN-JETSON-NANO-MASKCHECK-USE-YOLOV4/blob/0b9cd81e17ecd3def6d169857d45b97a253f8d37/gif/test1.gif)   


