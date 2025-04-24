# How to get?

Please visit https://www.insta360.com/sdk/apply to apply for the latest SDK.

# Support

Developers' Page: https://www.insta360.com/developer/home

Insta360 Enterprise: https://www.insta360.com/enterprise

Issue Report: https://insta.jinshuju.com/f/hZ4aMW

# [中文文档](README_zh.md)

# **Overview**

MediaSDK is mainly used for stitching panoramic materials. The currently supported device models include: **ONE X, ONE R/RS** (Standard fisheye and one-inch fisheye), **ONE X2, X3, X4, X5** cameras for panoramic materials. This SDK supports **video export** and **image export**. The main supported platforms are **Windows** and **Ubuntu 22.04**. For specific API usage, please refer to MediaSDK/example/main.cc.

- **Supported cameras**

| Model                                    | Link                                                      |
| :--------------------------------------- | :-------------------------------------------------------- |
| ONE X (Discontinued)                     | http://insta360.com/product/insta360-onex/                |
| ONE R Twin Edition (Discontinued)        | http://insta360.com/product/insta360-oner_twin-edition    |
| ONE X2                                   | https://www.insta360.com/product/insta360-onex2           |
| ONE RS 1-Inch 360 Edition (Discontinued) | https://www.insta360.com/product/insta360-oners/1inch-360 |
| X3                                       | https://www.insta360.com/product/insta360-x3              |
| X4                                       | https://www.insta360.com/product/insta360-x4              |
| X5                                       | https://www.insta360.com/product/insta360-x5              |

- **Supported platforms**

| Platform | Version                                                      |
| :------- | :----------------------------------------------------------- |
| Windows  | Windows 7 or later, only x64 supported                       |
| Linux    | Ubuntu 22.04 (x86_64), other distributions need to be tested |

- **Supported file format**

| filetype | import format | export format |
| :------- | :------------ | :------------ |
| Video    | insv          | mp4           |
| Image    | insp/jpeg     | jpg           |

# **Function Overview**

# **Notes**

The SDK requires that all file path strings must be encoded in UTF-8.

# **API Description**

##  **Initialize SDK Environment**
void InitEnv() (GPU Version)

This API must be called at the start of the main program to initialize the SDK environment.

##  **Input and Output Parameter Settings**

### **Input Path: void SetInputPath(std::vector\<std::string>& input\_paths)**

This API is used to set the input paths of the materials. It is an array and is valid for both videos and photos.

For videos, this array typically contains at most two material files. Materials with a resolution of **5.7K or higher** require two material files as input (except for materials captured with X4 cameras). For **X4 cameras**, dual video track storage is currently used. Regardless of resolution, there is **only one original video file**.

**Example Usage**

```c++
// For dual-track 5.7K materials
std::string _00_file = "/path/to/VID_XXX_..._00_XXX.insv";
std::string _10_file = "/path/to/VID_XXX_..._10_XXX.insv"
std::vector<std::string> input_path = {_00_file,_10_file};
videoStitcher->SetInputPath(input_path);

// For single-track material files
std::string insv_file = "/path/to/VID_XXX_..._00_XXX.insv";
std::vector<std::string> input_path = {insv_file};
videoStitcher->SetInputPath(input_path);
```

For **photo files**, this array can accept multiple inputs **(but not exactly 2)**. If **3, 5,  7 or 9 materials** are input, they are assumed to be **HDR photos**, and **HDR fusion** will be applied automatically. For **X4 cameras**, the **default HDR materials** captured by the camera **have already undergone in-camera HDR fusion**. Therefore, only **one material file** is output from the camera.



### **Output Path: void SetOutputPath(const std::string& output\_path)**

This API is used to set the output path. It is valid for both video and photo outputs. The parameter should be a full path.

**Supported Output Formats:**

> For videos, the path should end with .mp4  **Example:**/output/to/path/video.mp4

> For images, the path should end with .jpg  **Example:** /output/to/path/image.jpg



### **Output Resolution: void SetOutputSize(int width, int height)**

This API is used to set the output resolution. The parameter width:height must have a **2:1 ratio**.

For the **CPU version of the SDK**, if a resolution that is too small is set and **moiré patterns** appear, you can use the **EnableAreaSample** API to **eliminate moiré effects**.



### **Encoding Format: void EnableH265Encoder()**

This API is used to set the encoding format to **H.265**. The default encoding format is **H.264**.&#x20;

When the output resolution is **greater than 4K**, it is **recommended to use H.265 encoding**, as **H.265 encoding supports hardware acceleration**, which can **speed up the export process**.



### **Output Bitrate: void SetOutputBitRate(int64\_t bitRate)**

This API is used to set the bitrate for video output. The unit is **bps**.

If not set, the original video bitrate will be used by default.

> **Example:**
>
> To output at **60 Mbps**, set the value as follows: bitRate = 60 × 1000 × 1000



### **Export Video as Image Sequences: void SetImageSequenceInfo(const std::string output\_dir, IMAGE\_TYPE image\_type)**

This function allows users to export video frames as an image sequence and configure the output path and image format.

**Parameters: output\_dir**

This parameter specifies the **directory-level** output path, **excluding file information**.

Before using this function, make sure that the **target directory** **has already been created**.

> **Example:&#x20;**/path/to/image\_save\_dir

The exported image files are **named based on the video frame timestamp (ms).**

> **Example:&#x20;**/path/to/image\_save\_dir/100.jpg *(This means the image was saved at the 100 ms video frame timestamp.)*

**Note:&#x20;**&#x69;mage\_type specifies the image format to be used. This can be either .png or .jpg.

**Note:&#x20;**&#x49;f you have used SetOutputPath, this does not need to be set again.



### **Export Selected Frames from Video: void SetExportFrameSequence(const std::vector\<uint64\_t>& vec)**

This API is used to export images from **specific video frame indices**. This function must be used together with **SetImageSequenceInfo**. The output image file names will be based on the **video frame index**.The **video frame index starts from 0**.

> **Example:&#x20;**/path/to/image\_save\_dir/10.jpg (this means the image is saved for video frame index **10**)

**Demo Example:**

```c++
// This sample code demonstrates extracting frames 0, 10, 20, and 30 from a video file
std::vector<uint64_t> seq_nos = {0，10，20，30};
const std::string image_seq_export_dir = /path/to/image_seq_dir;
const IMAGE_TYPE image_format = IMAGE_TYPE::JPEG;
...
videoStitcher->SetExportFrameSequence(seq_nos);
videoStitcher->SetImageSequenceInfo(image_seq_export_dir,image_format);
...
videoStitcher->StartStitch()
```



## **Stabilization Parameter Settings**

### **Enable Stabilization: void EnableFlowState(bool enable)**

This API is used to configure the stabilization option, determining whether to enable stabilization.



### **Enable Direction Lock: void EnableDirectionLock(bool enable)**

This API is used to enable **direction lock**.



## **Stitching Parameter Settings**

### **Stitching Type: void SetStitchType(STITCH\_TYPE type)**

This API is used to set the stitching type. The available stitching types are as follows:

```plain&#x20;text
enum class STITCH_TYPE {
    TEMPLATE,       // Template stitching
    OPTFLOW,        // Optical flow stitching
    DYNAMICSTITCH,  // Dynamic stitching
    AIFLOW          // AI stitching
};
```

**Usage Scenarios**

* **Template Stitching**: An older stitching algorithm that provides poor stitching results for near-field scenes, but is fast and has low computational cost.

* **Dynamic Stitching**: Suitable for scenes containing motion or situations with rapid changes in movement.

* **Optical Flow Stitching**: Similar in function to dynamic stitching but optimized for higher accuracy.

* **AI Stitching**: Based on Insta360’s optimized optical flow stitching technology, offering superior stitching results.

> **Computational Cost vs. Stitching Quality**
>
> **AI Stitching > Optical Flow Stitching > Dynamic Stitching > Template Stitching**

> **Stitching Speed**
>
> **Template Stitching > Dynamic Stitching > Optical Flow Stitching > AI Stitching**

**Note:**

When using **AI Stitching**, you must call the **SetAiStitchModelFile** API to specify the model file. If this step is skipped, the stitching settings will be **invalid**.



### **AI Stitching Model: void SetAiStitchModelFile(const std::string& model\_file)**

This API is used to set the AI stitching model, which is required for AI stitching.

> **Model File Example:&#x20;**&#x4D;ediaSDK/modelfile/ai\_stitch\_model.ins



### **Chromatic Calibration: void EnableStitchFusion(bool enable)**

This API is used to enable Chromatic Calibration.



**Causes of Chromatic Aberration:&#x20;**&#x53;ince the two lenses operate separately, the video exposure captured by each lens may be slightly different. When stitched together, **noticeable brightness differences** may appear. Additionally, due to different lighting conditions on each side of the lens, the camera exposure may vary. Sometimes, the front lens captures a significantly different brightness level compared to the rear lens. This effect is especially prominent in areas with **high contrast lighting conditions**.

The chromatic calibration function is developed to address such issues.



### **Lens Guard: void SetCameraAccessoryType(CameraAccessoryType type)**

This API is used to **set the lens guard type**. If a lens guard is used during shooting, it must also be specified when stitching. Otherwise, the stabilization effect may be incorrect.



The following are the available **lens guard types**:

```c++
enum class CameraAccessoryType {
    kNormal = 0,            
    kWaterproof,            // Waterproof case (one/onex/onex2/oner/oners/onex3)
    kOnerLensGuard,         // Adhesive lens guard (oner/oners)
    kOnerLensGuardPro,      // Clip-on lens guard (oner/oners)
    kOnex2LensGuard,        // Adhesive lens guard (oner/oners/onex2/onex3)
    kOnex2LensGuardPro,     // Clip-on lens guard (onex2)
    k283PanoLensGuardPro,   // Clip-on lens guard for 283 panoramic lens (oner/oners)
    kDiveCaseAir,           // Dive case (above water) (onex/onex2/oner/oners/onex3)
    kDiveCaseWater,         // Dive case (underwater) (onex/onex2/oner/oners/onex3)
    kInvisibleDiveCaseAir,  // Invisible Dive Case  (Above water) (X3/X4)
    kInvisibleDiveCaseWater,// Invisible Dive Case  (underwater) (X3/X4)
    kOnex4LensGuardA,       // X4 A-grade plastic lens guard
    kOnex4LensGuardS,       // X4 S-grade glass lens guard
    kOnex3LensGuardA,       // X3 A-grade plastic lens guard
    kOnex3LensGuardS        // X3 S-grade glass lens guard
};
```

> Lens Guard Classification
>
> Standard lens guards in the store are classified as A-grade, while  Premium guards are classified as S-grade.



## **Image Setting Parameters**

### **Color Plus: void EnableColorPlus(bool enable, const std::string& model\_path)**

This API is used to enable or disable the **Color Plus** function. This is an **AI-based feature**, requiring the path to an **AI model** to be set.

> **Model file:** SDK_ROOT_DIR/data/colorplus_model.ins

### **Denoise: void EnableSequenceDenoise(bool enable)**

This API is used to **enable or disable** the denoising feature.

**Video denoise&#x20;**&#x65;mploy&#x73;**&#x20;image processing techniques** and is based on **multi-frame denoising** to reduce or eliminate noise in videos. Compared to **single-frame denoising**, multi-frame denoising **leverages redundant information from multiple frames**, but it also **requires more computing resources** and **slows down export speed**.

For **image materials**, the **model file path** should be specified.

> **Model file:** SDK_ROOT_DIR/data/jpg_denoise_9d006262.ins

### Remove purple edge: void EnableDefringe(bool enable, const std::string& defringe_model_path)

This interface is used to eliminate the purple edge phenomenon caused by lighting during recording, such as outdoor strong light and indoor lighting scenes.

> **Model file:** SDK_ROOT/modelfile/defringe_hr_dynamic_7b56e80f.ins

### Remove strobe: void EnableDeflicker(bool enable, const std::string& deflicker_model_path)

This interface is used to eliminate screen flickering problems caused by lighting during recording.

> **Model file:** SDK_ROOT/modelfile/deflicker_86ccba0d.ins

## **Stitching Process**

### **Stitching Progress Callback: void SetStitchStateCallback(stitch\_error\_callback callback)**

This API is primarily used for **stitching status and progress notifications**.&#x20;

It is **recommended not to perform time-consuming operations** within this callback, as this may **affect stitching speed**.

**Example Code:**

```c++
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



### **Stitching Error Callback: void SetStitchProgressCallback(stitch\_process\_callback callback)**

This API is used to receive error messages during the stitching process.

**Example Code:**

```c++
video_stitcher->SetStitchStateCallback([&](int error, const char* errinfo) {
    std::cout << "error: " << errinfo << std::endl;
    has_error = true;
    cond_.notify_one();
});
```



### **Start Stitching: void StartStitch()**

This API is used to **start the stitching process**.

**Note:**

Ensure that all **parameter settings are completed before calling this API**.

If this API is executed **before setting the necessary parameters**, the parameters **will not take effect**.



### **Cancel Stitching: bool CancelStitch()**

This API is used to **interrupt the stitching process**.



### **Get Stitching Progress: int GetStitchProgress() const**

This API is used to **retrieve the stitching progress**.



## **Logging Functionality**

###  **Set Log Path: void SetLogPath(const std::string log\_path)**

This API is primarily used to set the **log file path** in the SDK, allowing SDK log information to be saved.

### **Set Log Print Level: void SetLogLevel(InsLogLevel level)**

This API is used to **set the logging level** within the SDK.



## **Hardware Codec Acceleration**

## **Live stream preview stitching**

###  Environmental preparation

 Camera stitch preview is based on CameraSDK and MediaSDK together. Header File is located in `MediaSDK_ROOT/include/ins_realtime_stitcher.`

 The main function of CameraSDK is to provide stitching parameters, video data, anti-shake data, and exposure data.

 The main function of MediaSDK is to use the parameters and data provided by cameraSDK to stitch together images and generate a 2:1 panoramic image.

###  Preview parameter acquisition and setting

```c++
#include <ins_realtime_stitcher.h>
// This interface mainly obtains the parameters required by mediaSDK
//...
// Cam is the current camera instance object
auto preview_param = cam->GetPreviewParam();

// Create a stitching instance object
auto stitcher = std::make_shared<ins::RealTimeStitcher>();

// Set preview parameters for stitching instance objects
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

###  Preview stream processing of original data source

```c++
// In the cameraSDK, you need to use the inheritance ins_camera :: StreamDelegate interface to achieve real-time data acquisition of the camera.
// Real-time data from cameraSDK can be transmitted to MediaSDK through demo examples

class StitchStreamDelegate : public ins_camera::StreamDelegate {
public:
    StitchDelegate(const std::shared_ptr<ins::RealTimeStitcher>& stitcher) :stitcher_(stitcher) {
    }

    virtual ~StitchDelegate() {
    }

    void OnAudioData(const uint8_t* data, size_t size, int64_t timestamp) override {}

    // Video data
    void OnVideoData(const uint8_t* data, size_t size, int64_t timestamp, uint8_t streamType, int stream_index) override {
        stitcher_->HandleVideoData(data, size, timestamp, streamType, stream_index);
    }
     
    // Anti-shake data
    void OnGyroData(const std::vector<ins_camera::GyroData>& data) override {
        std::vector<ins::GyroData> data_vec(data.size());
        memcpy(data_vec.data(), data.data(), data.size() * sizeof(ins_camera::GyroData));
        stitcher_->HandleGyroData(data_vec);
    }
 
    // Exposure data
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

###  Set preview parameters

####   stitching type

  Reference 4.4.1 and 4.4.2.

####   Stabilization parameter

  Parameter 3.

####   Protective mirror

  Parameters 4.4.4.

####   Output screen size

  For output size, if not set, the output size is the resolution of the current preview.

  If the performance output frame rate, the resolution size can be reduced.

### Get stitched data

 The currently supported format for stitched data is RGBA.

 You can get the stitched video picture by setting `SetStitchRealTimeDataCallback` this callback interface. It is recommended not to perform time-consuming operations in this callback. The reference code is as follows:

```c++
stitcher->SetStitchRealTimeDataCallback([&](uint8_t* data[4], int linesize[4], int width, int height, int format, int64_t timestamp) {
        show_image_ = cv::Mat(height, width, CV_8UC4, data[0]).clone();
    });
```

###  Enable preview

```c++
// Set up the delegation interface for camera real-time data
std::shared_ptr<ins_camera::StreamDelegate> delegate = std::make_shared<StitchStreamDelegate>(stitcher);
cam->SetStreamDelegate(delegate);
ins_camera::LiveStreamParam param;
//...
// Open the preview of the camera
if (cam->StartLiveStreaming(param)) {
// Start the stitching process
    stitcher->StartStitch();
    std::cout << "successfully started live stream" << std::endl;
}
```

###  Close preview

```c++
// Close camera preview stream
if (cam->StopLiveStreaming()) {
// Stop the stitching process
    stitcher->CancelStitch();
    std::cout << "success!" << std::endl;
}
```

## **Error Codes**

| Error Code                | Error Message                                                |
| ------------------------- | ------------------------------------------------------------ |
| E_OPEN_FILE(1)            | Failed to open file                                          |
| E_PARSE_METADATA(2)       | Failed to parse file metadata                                |
| E_CREATE_OFFSCREEN(3)     | Common offscreen rendering failure                           |
| E_CREATE_RENDER_MODEL(4)  | Failed to create render model                                |
| E_FRAME_PARSE(5)          | Failed to retrieve data frame                                |
| E_CREATE_RENDER_SOURCE(6) | Failed to create rendering data source                       |
| E_UPDATE_RENDER_SOURCE(7) | Failed to update rendering data frame                        |
| E_RENDER_FRAME(7)         | Failed to render frame                                       |
| E_SAVE_FRAME(8)           | Failed to save image                                         |
| E_VIDEO_FRAME_EXPORTOR(9) | Failed to create video frame exporter                        |
| E_UNKNOWN(999)            | Unknown error, please provide detailed information for analysis |

