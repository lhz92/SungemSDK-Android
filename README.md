
# SungemSDK-Android API
The current version is 0.1.1
## ConnectStatus 状态参数
  - HS_OK ：正常
  - HS_BUSY ：队列正忙
  - HS_ERROR ：通信异常
  - HS_NO_FILE ：没有索引到文件
  - HS_UNSUPPORTED_GRAPH_FILE ：不支持的graph文件
  - STATUS_WAIT_TIMEOUT ： 等待命令超时时间
---
## HsBaseThread
HsBaseThread is a thread for HornedSungem interacion, developers can extend this class for function expansion.
### 类内主要变量：
  -  mHsApi:与角蜂鸟通信的主要api类，可调用类里对应函数执行相应操作

### 构造器
 - ConnectBridge：在usb连接上执行回调函数openSucceed时的参数，用于连接Usb设备与当前类
 - zoom : 决定获取角蜂鸟自带摄像头图像的分辨率大小，true为640\*360的图像，false为1920\*1080的图像

### Public Member Functions

| 返回值    | 函数名  | 描述 |
| :----- | :------ | :----------: |
| int | [allocateGraph(String filename)](#allocategraphstring-filename) | 分配卷积神经网络模型到角蜂鸟，传入文件路径|
| int | [allocateGraphByAssets(String filename)](#jump2) | 分配卷积神经网络模型到角蜂鸟，传入文件在assets下路径 |
| byte[] | [getImage(float stdValue,float mean)](#jump3) | 获取graph图像 |
| byte[] | [deviceGetImage()](#jump4) | 获取设备图像 |
| void | [setZoom(boolean zoom)](#jump5) | 设置摄像头分辨率 |
| int | [loadTensor(float[] data,int length,int parameter)](#jumm6) | 角蜂鸟加载预处理后的图像数据 |
| int | [loadTensor(byte[] data,int length,int parameter)](#jump6) | 角蜂鸟加载经过预处理后的数据 |
| byte[] | [getResult(int parameter)](#jump7) | retrieve the inference result |
| void | [close()](#jump8) | 关闭设备 |
1. **allocateGraph(String filename)**：此函数是分配一个神经网络模型给角蜂鸟，通过加载该模型来实现某个功能

  - 参数 Parameters：
    - filename（String类型） 文件绝对路径    
  - 返回 return:返回操作状态 （状态声明都在HsConnectApi里）
    - HS_OK值为0，表示成功
    - HS_ERROR值为-2，表示加载失败
    - HS_UNSUPPORTED_GRAPH_FILE值为-10,表示不支持的模型文件
    - HS_NO_FILE值为-12，表示filename路径有误，没有文件
  - Example用法:

    <pre><code>int status=allocateGraph(Environment.getExternalStorageDirectory().getAbsolutePath() + "/hs/" + "graph_face_SSD");
    </code></pre>
2. <span id="jump2" >**allocateGraphByAssets(filename)**</span> :与上述方法功能相似，区别在于参数表示在android工程assets包下的文件   
  - Example用法:

     <pre><code>int status = allocateGraphByAssets("graph_face_SSD");
     </code></pre>

3. <span id="jump3">**getImage(stdValue,meanValue)**</span>:获取graph的图像
  - 参数 Parameters：预处理的值，根据卷积神经网络的区别，参数的值不同，具体参照[模型列表](https://hornedsungem.github.io/Docs/cn/model/)
    - stdValue（float类型）
    - meanValue（float类型）
  - 返回 return:返回 byte[] ，表示该图像的原始数据

    - 如果参数zoom为true: byte[]大小为640\*360\*3，得到bgr的图像

    - 如果参数zoom为false: byte[]大小为1920\*1080\*3 rgb图像顺序排列，需要作对应转化才能显示正确，转化代码如下：

      <pre><code>
      byte[] bytes = mHsApi.getImage(STD, MEAN);
      int FRAME_W = 1920;
      int FRAME_H = 1080;
      byte[] bytes_rgb = new byte[FRAME_W * FRAME_H * 3];
      for (int i = 0; i < FRAME_H * FRAME_W; i++) {
        bytes_rgb[i * 3 + 2] = bytes[i];//r
        bytes_rgb[i * 3 + 1] = bytes[FRAME_W * FRAME_H + i];//g
        bytes_rgb[i * 3] = bytes[FRAME_W * FRAME_H * 2 + i];//b
      }
      opencv_core.IplImage bgrImage = opencv_core.IplImage.create(FRAME_W, FRAME_H, opencv_core.IPL_DEPTH_8U, 3);
      bgrImage.getByteBuffer().put(bytes_rgb);
      </code></pre>

4. <span id="jump4">**deviceGetImage()**</span>:从设备摄像头里取出图像
  - 返回 return:返回图像byte[],表示图像的原始数据，**返回值跟getImage()函数相似，此处就不具体阐述**

5. <span id="jump5">**setZoom( zoom)**</span>：设置获取角蜂鸟的图像分辨率大小，用于处理得到数据，设置图像宽高等
  - 参数 zoom(boolean类型):
    - true:设置为true,获取分辨率为640\*360
    - false:设置为false,获取分辨率为1920\*1080

6. <span id="jump6">**loadTensor(inputTensor,length,userObj)**</span>：把外部的图像或数据传送给角蜂鸟

  **该方法重载，可根据开发者使用参数不同选择对应函数**

  - 参数 Parameters：
    - inputTensor（float[]或byte[]类型）：传送的数据
    - length(int类型):传送数据数组的大小，默认数组index从0开始
    - userObj(int类型)：**用户配置参数，当前版本未使用，可随意填写**

  - 返回 return:返回该函数执行后状态
    - HS_ERROR:读写发生异常  
    - HS_BUSY:当前设备正忙，操作失败
    - HS_OK:正常

  - Example：此处采用hello2018的示例代码

    <pre><code>
    for (int i = 1; i < 5; i++) {
        int[] ints = new int[28 * 28];
        try {
            InputStream inputStream = mActivity.getAssets().open("hello/" + i + ".jpg");
            Bitmap bitmap = BitmapFactory.decodeStream(inputStream);
            bitmap.getPixels(ints, 0, 28, 0, 0, 28, 28);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        float[] float_tensor = new float[28 * 28];
        for (int j = 0; j < 28 * 28; j++) {
            float_tensor[j] = Color.red(ints[j]) * 0.007843f - 1;
        }
        int status_load = mHsApi.loadTensor(float_tensor, float_tensor.length, 0);
    }
    </code></pre>

7. <span id="jump7" >**getResult(userObj)**</span>:获取图像处理后的返回结果
  - 参数 Parameters：
    - userObj(int类型)：**用户配置参数，当前版本未使用，可随意填写**
  - 基本用法 Example：

      ```
      float[] result = mHsApi.getResult(0);
      ```
  - 返回 return：返回float[] 根据使用神经网络不同，返回数组长度和解析也不尽相同
    1. 人脸及物体检测返回结果处理：

      <pre><code>    
      //结果处理
      int num = (int) floats[0];//第一个数为检测到的个数
      if (num > 0) {
          for (int i = 0; i < num; i++) {
              int type = (int) (floats[7 * (i + 1) + 1]);//类别
              int x1 = (int) (floats[7 * (i + 1) + 3] * FRAME_W);
              int y1 = (int) (floats[7 * (i + 1) + 4] * FRAME_H);
              int x2 = (int) (floats[7 * (i + 1) + 5] * FRAME_W);
              int y2 = (int) (floats[7 * (i + 1) + 6] * FRAME_H);
              int wight = x2 - x1;
              int height = y2 - y1;
              //如果有值不满足条件，这组数据干掉
              int percentage = (int) (floats[7 * (i + 1) + 2] * 100);//置信度
              if (percentage <= MIN_SCORE_PERCENT) {
                continue;
              }
              if (wight >= FRAME_W * 0.8 || height >= FRAME_H * 0.8) {
                continue;
              }
              if (x1 < 0 || x2 < 0 || y1 < 0 || y2 < 0 || wight < 0 || height < 0) {
                continue;
              }
          }
      }
      </code></pre>
    2. 数字识别返回结果处理：返回10个float数，分别对应0-9的置信度
    3. 简笔画识别返回结果处理：返回345个float值，分别对应345种物体的置信度，详情可见[手绘识别](https://hornedsungem.github.io/Docs/workflow/)文档
8. <span id="jump8" >**close()**</span>: 线程关闭时调用，执行操作包括：deallocate模型,关闭角蜂鸟设备等
  - 无参数
  - Example：示例代码

    <pre><code>
    if (mHsThread != null) {
       mHsThread.setRunning(false);
       mHsThread.close();
     }
    </code></pre>       
---

## HsBaseActivity相关属性及函数
该抽象类继承Activity，定义几个抽象回调函数用于各种行为处理，是开发者使用角蜂鸟的基础Activity

### 相关回调

| 返回值  | 函数名 | 描述 |
| :----- | :------ | :---------- |
| void | [openSucceed(ConnectBridge connectBridge)](#jump11) | 搜索到角蜂鸟设备，并连接成功|
| void | [openFailed()](#jump12) | 角蜂鸟连接失败或未授予权限 |
| void | [disConnected()](#jump13) | 断开角蜂鸟连接|
1. <span id="jump11" >**openSucceed(connectBridge)**</span>: 打开角蜂鸟设备以后的回调函数，主要用于进行通信，比如创建角蜂鸟线程，给出提醒等
  - 参数 Parameters：
    - connectBridge: usb设备通信的连接桥
  - Example: 示例代码

    <pre><code>
    @Override
    public void openSucceed(UsbLinkVsc usbLinkVsc) {
        mTvTip.setVisibility(View.GONE);
        mHsThread = new FaceDetectionThread(FaceDetectorActivity.this, usbLinkVsc, mHandler);
        mHsThread.start();
    }
    </code></pre>

2. <span id="jump12" >**openFailed()**</span>:打开角蜂鸟设备失败以后的回调函数，主要用于失败以后执行对应操作，比如提醒重新插拔角蜂鸟等
  - 无参数
  - Example: 部分示例代码

    <pre><code>
    @Override
    public void openFailed() {
        mTvTip.setText("请重新插拔角蜂鸟允许权限");
  }
  </code></pre>

3. <span id="jump13" >**disConnected()**</span>:拔掉角蜂鸟或usb接触有问题的回调函数，用于处理断开后的release操作，比如停止线程等
  - 无参数
  - Example: 部分示例代码

    <pre><code>
    @Override
    public void disConnected() {
        Toast.makeText(this, "断开连接", Toast.LENGTH_SHORT).show();
        if (mHsThread != null) {
          mHsThread.setRunning(false);
          mHsThread.close();
        }
    }
    </code></pre>
