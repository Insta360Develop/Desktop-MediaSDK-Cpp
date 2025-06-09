# 文档概述

​        MediaSDK主要用于对全景素材进行拼接，目前支持的机型有ONE X、ONE R/RS(普通鱼眼和一英寸鱼眼)、ONE X2、X3、X4以及X5相机的全景素材。支持视频导出和图片导出。支持平台主要有Windows和Ubuntu 22.04平台。具体接口的使用可以参考SDK中example/main.cc.

- 支持机型

| 型号                                     | 连接                                                      |
| :--------------------------------------- | :-------------------------------------------------------- |
| ONE X (Discontinued)                     | http://insta360.com/product/insta360-onex/                |
| ONE R Twin Edition (Discontinued)        | http://insta360.com/product/insta360-oner_twin-edition    |
| ONE X2                                   | https://www.insta360.com/product/insta360-onex2           |
| ONE RS 1-Inch 360 Edition (Discontinued) | https://www.insta360.com/product/insta360-oners/1inch-360 |
| X3                                       | https://www.insta360.com/product/insta360-x3              |
| X4                                       | https://www.insta360.com/product/insta360-x4              |
| X5                                       | https://www.insta360.com/product/insta360-x5              |

- **支持平台**

| 平台    | 版本                                       |
| :------ | :----------------------------------------- |
| Windows | Windows 7 或更高版本，仅支持 x64           |
| Linux   | Ubuntu 22.04（x86_64），其他发行版需要测试 |

- **支持的文件格式**

| 文件类型 | 导入格式  | 导出格式 |
| :------- | :-------- | :------- |
| Video    | insv      | mp4      |
| Image    | insp/jpeg | jpg      |

# 目录
* [初始化SDK环境](#初始化sdk环境)
* [输入输出参数设置](#输入输出参数设置)
* [防抖参数设置](#防抖参数设置)
* [拼接参数设置](#拼接参数设置)
* [图像设置参数](#图像设置参数)
* [拼接流程](#拼接流程)
* [日记功能](#日记功能)
* [硬件编解码加速](#硬件编解码加速)
* [实时预览流拼接](#实时预览流拼接)
* [错误码](#错误码)

# 注意事项：

SDK中要求所有文件路径字符编码都是UTF-8编码

# 接口说明

## 初始化SDK环境
void InitEnv() (GPU版本);

这个接口必须在主程序main开始调用，用于初始化SDK环境

## 输入输出参数设置

### 输入路径: void SetInputPath(std::vector\<std::string>& input\_paths)

这个接口用于设置素材的输入路径，是一个数组。对于视频和照片都是生效的.

对于视频，这个数组最多是有两个素材文件。大于等于5.7k分辨率的素材需要输入两个素材文件(除了X4相机的素材，X4相机的素材，目前使用双视频轨道保存，不管是什么分辨率，只有一个素材文件)。

示例如下:

```c++
//双路的5.7K素材如下
std::string _00_file = "/path/to/VID_XXX_..._00_XXX.insv";
std::string _10_file = "/path/to/VID_XXX_..._10_XXX.insv"
std::vector<std::string> input_path = {_00_file,_10_file};
videoStitcher->SetInputPath(input_path);

//单路文件素材如下
std::string insv_file = "/path/to/VID_XXX_..._00_XXX.insv";
std::vector<std::string> input_path = {insv_file};
videoStitcher->SetInputPath(input_path);
```

对于图片，这个数组可以输入多个(不能输入2个). 如果输入3个、5个、7个或者9个素材，默认为是HDR照片会进行HDR融合。特别说明X4相机默认拍摄的HDR素材，已经是机内HDR融合好的素材。相机拍摄输出的素材文件只有一个。

### 输出路径: void SetOutputPath(const std::string& output\_path)

这个接口用于设置导出的路径。对于视频和照片都是生效的。这个参数是全路径的。

支持导出的格式：

> 对于视频，以.mp4结尾。 示例:/ouput/to/path/video.mp4

> 对于图片，以.jpg结尾。 示例: /output/to/path/image.jpg

### 输出分辨率: void SetOutputSize(int width, int height)

这个接口用于设置导出的分辨率。参数 width:height 必须是 2:1。

*针对CPU版本的SDK，如果设置的分辨率过于小出现的摩尔纹现象，可以通过这个接口**EnableAreaSample**去消除摩尔纹现象*

### 编码格式: void EnableH265Encoder()

这个接口用于设置编码格式为h265. 默认编码格式h264.  输出分辨率大于4K时，建议使用h265编码可以使用硬件编码，可以加快导出进度。

### 输出码率: void SetOutputBitRate(int64\_t bitRate)

这个接口用于设置导出视频的码率，单位是bps。如果不设置，默认会采用原视频的码率导出。

> 比如输出 60Mbps，输入的数值：bitRate = 60 X 1000 x 1000

### 图片序列导出信息: void SetImageSequenceInfo(const std::string output\_dir, IMAGE\_TYPE image\_type)

这个功能允许用户将视频原片导出为图像序列，并可以设置导出图像序列的路径和图片格式。

参数：output\_dir 这个参数设置路径是到目录级别的，不包含文件信息。使用这个功能之前需要确保目标目录已经创建好.

> 示例：/path/to/image\_save\_dir

输出的图片文件的命名是以视频帧的时间戳(ms)的。

> 示例：/path/to/image\_save\_dir/100.jpg(这个表示视频帧100ms时保存的图片名称).

参数:  image\_type 这个参数设置图片的导出格式。目前支持png和jpg。

注意：如果设置了这个接口，**&#x20;SetOutputPath&#x20;**&#x8FD9;个接口设置的参数是无效的。

### 指定图片序列导出: void SetExportFrameSequence(const std::vector\<uint64\_t>& vec)

这个接口用于将指定的视频帧序号序列导出成图片。这个功能是和**SetImageSequenceInfo**这个接口配合一起使用的。输出的图片文件的命名是以视频帧的序号的。视频帧序号以0开始。

> 示例：/path/to/image\_save\_dir/10.jpg(这个表示视频帧序号为10时保存的图片名称)

演示demo：

```c++
// 这个伪代码演示了从视频文件中抽取第0，10，20，30的视频帧进行拼接导出成图片
std::vector<uint64_t> seq_nos = {0，10，20，30};
const std::string image_seq_export_dir = /path/to/image_seq_dir;
const IMAGE_TYPE image_format = IMAGE_TYPE::JPEG;
...
videoStitcher->SetExportFrameSequence(seq_nos);
videoStitcher->SetImageSequenceInfo(image_seq_export_dir,image_format);
...
videoStitcher->StartStitch()
```

## 防抖参数设置

### 防抖开启: void EnableFlowState(bool enable)

这个接口用于设置防抖选项，是否开启防抖

### 方向锁定: void EnableDirectionLock(bool enable)

 这个接口用于开启方向锁定



## 拼接参数设置

### 拼接类型: void SetStitchType(STITCH\_TYPE type)

这个接口用于设置拼接类型。拼接可分为以下几种类型：

```c++
enum class STITCH_TYPE {
    TEMPLATE,       // 模板拼接
    OPTFLOW,        // 光流拼接
    DYNAMICSTITCH,  // 动态光流拼接
    AIFLOW          // ai拼接
};
```

使用场景

- 模板拼接：比较老的拼接算法，对近景拼接效果不好，但是速度快，性能消耗低
- 动态拼接：适合包含近景的场景，或者有运动和快速变化的情况
- 光流拼接：使用场景和动态拼接相同
- AI 拼接：基于影石 Insta360 现有的光流拼接技术的优化算法，提供更优的拼接效果

> 性能消耗及拼接效果：AI 拼接>光流拼接>动态拼接>模板拼接

> 拼接速度：模板拼接>动态拼接>光流拼接>AI 拼接

注意: 使用AI拼接时,必须使用接口 **SetAiStitchModelFile** 将模型文件设置进去,否则设置拼接效果失效

### AI拼接模型: void SetAiStitchModelFile(const std::string& model\_file)

这个接口用于设置AI拼接的模型,配合AI拼接使用

> 模型文件v1：SDK_ROOT_DIR/data/ai_stitch_model_v1.ins
> 
> 模型文件v2：SDK_ROOT_DIR/data/ai_stitch_model_v2.ins

对于这个X4相机之前的素材，使用v1版本的模型文件。对于X5相机的素材，使用v2版本的模型文件

### 消色差: void EnableStitchFusion(bool enable)

这个接口用于开启消色差的功能

产生色差的原因：两个镜头是分开的，得出的视频曝光可能不太一致，当把它们拼接在一起的时候，会有比较明显的亮度差；另外，因为镜头两边的光照不一样，相机曝光不同，有时候前后镜头拍出来的画面也会有明显的亮度差，这种现象在光差比大的地方尤其明显，消色差就是为了解决此类问题而开发

### 保护镜: void SetCameraAccessoryType(CameraAccessoryType type)

这个接口用于设置保护镜. 如果在拍摄过程中,相机佩戴了保护镜. 需要在拼接的时候也设置保护镜的类型,以防拼接效果不对. 下面保护镜的类型&#x20;

```c++
enum class CameraAccessoryType {
    kNormal = 0,
    kWaterproof,           // (one/onex/onex2/oner/oners/onex2/onex3) 潜水壳
    kOnerLensGuard,        // (oner/oners) 黏贴式保护镜
    kOnerLensGuardPro,     // (oner/oners) 卡扣式保护镜
    kOnex2LensGuard,       // (oner/oners/onex2/onex3) 黏贴式保护镜
    kOnex2LensGuardPro,    // (onex2)卡扣式保护镜
    k283PanoLensGuardPro,  // (oner/oners) 283全景镜头的卡扣式保护镜
    kDiveCaseAir,          // (onex/onex2/oner/oners/onex2/onex3) 潜水壳(水上)
    kDiveCaseWater,        // (onex/onex2/oner/oners/onex2/onex3) 潜水壳(水下)
    kInvisibleDiveCaseAir,  // X3/X4 全隐形潜水壳(水上)
    kInvisibleDiveCaseWater,  // X3/X4 全隐形潜水壳(水下)
    kOnex4LensGuardA,  //  X4 A级塑胶保护镜
    kOnex4LensGuardS,  //  X4 S级玻璃保护镜
    kOnex3LensGuardA,  //  X3 A级塑胶保护镜
    kOnex3LensGuardS,  //  X3 A级玻璃保护镜
};
```

商城中的标准保护镜为A级，高级保护镜为S级

## 图像设置参数

### 色彩增强: void EnableColorPlus(bool enable, const std::string& model\_path)

这个接口用于设置是否开启色彩增强功能.这个AI功能,需要设置一下AI模型的路径

> 模型文件：SDK_ROOT_DIR/data/colorplus_model.ins

### 降噪: void EnableDenoise(bool enable, const std::string& model\_path)

这个接口用于设置是否开启降噪功能

视频中使用的是多帧降噪是通过图像处理技术减轻或去除视频中的噪点的过程。相比于单帧降噪，视频降噪常常利用前后多帧的冗余信息. 同时也比较消耗性能和减慢导出速度。
针对图片素材，需要输入模型文件的路径：

> 模型路径：SDK_ROOT_DIR/data/colorplus_model.ins

###  去紫边: void EnableDefringe(bool enable, const std::string& defringe_model_path)

  这个接口用于消除在录制过程中由于光照出现的紫边现象，如室外强光、室内灯光的场景。

> 模型路径为：SDK_ROOT/modelfile/defringe_hr_dynamic_7b56e80f.ins

###  去频闪: void EnableDeflicker(bool enable, const std::string& deflicker_model_path)

 这个接口用于消除在录制过程中由于光照出现的屏闪问题。

>模型路径为：SDK_ROOT/modelfile/deflicker_86ccba0d.ins

## 拼接流程

### 拼接进度回调: void SetStitchStateCallback(stitch\_error\_callback callback)

这个接口主要用于拼接状态和进度信息的通知, 使用方法如下. 同时建议不能在这个回调中使用耗时的操作,这样会影响到拼接速度.

```C++
video_stitcher->SetStitchProgressCallback([&](int process, int error) {
    if (stitch_progress != process) {
        std::cout << "\r";
        std::cout << "process = " << process << "%";
        std::cout << std::flush;
        stitch_progress = process;
    }

    if (stitch_progress == 100) {
        std::cout << std::endl;
        std::unique_lock<std::mutex> lck(mutex_);
        cond_.notify_one();
        is_finisned = true;
    }
 });
```

### 拼接错误回调: void SetStitchProgressCallback(stitch\_process\_callback callback)

这个接口用于接收拼接过程中错误信息的返回.

```C++
video_stitcher->SetStitchStateCallback([&](int error, const char* errinfo) {
    std::cout << "error: " << errinfo << std::endl;
    has_error = true;
    cond_.notify_one();
});
```

### 开启拼接: void StartStitch()

这个接口开启拼接流程

注意在上面的参数设置完成后，才去执行这个接口。不允许在这个接口执行以后，设置上面的参数，参数不生效。

### 取消拼接: bool CancelStitch()

这个接口中断拼接流程

### 获取拼接进度: int GetStitchProgress() const

这个接口用于获取拼接进度

## 日记功能

### 设置日记路径:  void SetLogPath(const std::string log\_path)

这个接口主要用于设置SDK中的日记路径，可以保存SDK中的日记信息

### 设置日记打印等级:  void SetLogLevel(InsLogLevel level)

这个接口用于设置SDK中日记打印的等级

## 硬件编解码加速

## 实时流预览拼接

###  环境准备

相机拼接预览的功能是基于CameraSDK和MediaSDK一起实现的。头文件位于**MediaSDK_ROOT/include/ins_realtime_stitcher.h。** 

**CameraSDK**的主要功能是提供拼接参数、视频数据、防抖数据以及曝光数据。

**MediaSDK**的主要功能使用cameraSDK提供的参数以及数据进行画面的拼接，生成一个2:1的全景画面。

可以参考MediaSDK_ROOT/example/realtime_stitcher_demo.cc

### 预览参数获取和设置

```c++
#include <ins_realtime_stitcher.h>
// 这个接口主要获取mediaSDK需要的参数
//...
// cam为当前相机实例对象
auto preview_param = cam->GetPreviewParam();

// 创建拼接实例对象
auto stitcher = std::make_shared<ins::RealTimeStitcher>();

// 设置预览参数给拼接实例对象
ins::CameraInfo camera_info;
camera_info.cameraName = preview_param.camera_name;
camera_info.decode_type = static_cast<ins::VideoDecodeType>(preview_param.encode_type);
camera_info.offset = preview_param.offset;
auto window_crop_info = preview_param.crop_info;
camera_info.window_crop_info_.crop_offset_x = window_crop_info.crop_offset_x;
camera_info.window_crop_info_.crop_offset_y = window_crop_info.crop_offset_y;
camera_info.window_crop_info_.dst_width = window_crop_info.dst_width;
camera_info.window_crop_info_.dst_height = window_crop_info.dst_height;
camera_info.window_crop_info_.src_width = window_crop_info.src_width;
camera_info.window_crop_info_.src_height = window_crop_info.src_height;
camera_info.gyro_timestamp = preview_param.gyro_timestamp;

stitcher->SetCameraInfo(camera_info);
```

###  预览流原始数据的处理

```c++
// 在cameraSDK中，要使用继承 ins_camera::StreamDelegate 接口才可以实现相机实时数据的获取。
// 可以在通过demo的示例将cameraSDK的实时数据传输给MediaSDK

class StitchStreamDelegate : public ins_camera::StreamDelegate {
public:
    StitchDelegate(const std::shared_ptr<ins::RealTimeStitcher>& stitcher) :stitcher_(stitcher) {
    }

    virtual ~StitchDelegate() {
    }

    void OnAudioData(const uint8_t* data, size_t size, int64_t timestamp) override {}

    // 视频数据
    void OnVideoData(const uint8_t* data, size_t size, int64_t timestamp, uint8_t streamType, int stream_index) override {
        stitcher_->HandleVideoData(data, size, timestamp, streamType, stream_index);
    }
     
    // 防抖数据
    void OnGyroData(const std::vector<ins_camera::GyroData>& data) override {
        std::vector<ins::GyroData> data_vec(data.size());
        memcpy(data_vec.data(), data.data(), data.size() * sizeof(ins_camera::GyroData));
        stitcher_->HandleGyroData(data_vec);
    }
 
    // 曝光数据
    void OnExposureData(const ins_camera::ExposureData& data) override {
        ins::ExposureData exposure_data{};
        exposure_data.exposure_time = data.exposure_time;
        exposure_data.timestamp = data.timestamp;
        stitcher_->HandleExposureData(exposure_data);
    }

private:
    std::shared_ptr<ins::RealTimeStitcher> stitcher_;
};
```

###  设置预览参数

####   拼接类型

  参考4.4.1以及4.4.2

####   防抖参数

  参数3

####   保护镜

  参数4.4.4

####   输出画面大小

  对于输出大小，如果不进行设置，输出的大小为当前预览的分辨率

  如果性能输出帧率，可以降低分辨率大小

### 获取拼接好的数据

拼接好的数据目前支持的格式是RGBA

可以通过设置**SetStitchRealTimeDataCallback** 这个回调接口获取到拼接好的视频画面，建议不要再这个回调里面执行耗时的操作。参考代码如下：

```c++
stitcher->SetStitchRealTimeDataCallback([&](uint8_t* data[4], int linesize[4], int width, int height, int format, int64_t timestamp) {
        show_image_ = cv::Mat(height, width, CV_8UC4, data[0]).clone();
    });
```

###  开启预览

```c++
// 设置相机实时数据的委托接口
std::shared_ptr<ins_camera::StreamDelegate> delegate = std::make_shared<StitchStreamDelegate>(stitcher);
cam->SetStreamDelegate(delegate);
ins_camera::LiveStreamParam param;
//...
// 开启相机的预览
if (cam->StartLiveStreaming(param)) {
// 开启拼接流程
    stitcher->StartStitch();
    std::cout << "successfully started live stream" << std::endl;
}
```

###  关闭预览

```c++
//关闭相机预览流
if (cam->StopLiveStreaming()) {
// 停止拼接流程
    stitcher->CancelStitch();
    std::cout << "success!" << std::endl;
}
```

## 错误码

| 错误码                          | 错误信息            |
| ---------------------------- | --------------- |
| E\_OPEN\_FILE(1)             | 打开文件失败          |
| E\_PARSE\_METADATA(2)        | 解析文件尾失败         |
| E\_CREATE\_OFFSCREEN(3)      | 常见离屏渲染失败        |
| E\_CREATE\_RENDER\_MODEL(4)  | 创建渲染模型失败        |
| E\_FRAME\_PARSE(5)           | 获取数据帧失败         |
| E\_CREATE\_RENDER\_SOURCE(6) | 创建渲染数据源失败       |
| E\_UPDATE\_RENDER\_SOURCE(7) | 更新数据帧到渲染源失败     |
| E\_RENDER\_FRAME(7)          | 渲染数据失败          |
| E\_SAVE\_FRAME(8)            | 保存图片失败          |
| E\_VIDEO\_FRAME\_EXPORTOR(9) | 创建视频提取器失败       |
| E\_UNKNOWN(999)              | 未知信息，需要提供详细信息分析 |

