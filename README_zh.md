# Insta360 OSC 使用文档
### 概览
**本文档基于[Google OSC](https://developers.google.com/streetview/open-spherical-camera)规范编写**
使用Insta360 ONE X、ONE X2、ONE R、ONE RS、X3、X4、X5时，设备连接到相机WiFi热点，此时相机的IP地址：192.168.42.1
**强烈建议** ： 所有 `osc/commands` 命令收到回复之后再发送下一条

- 发送请求时请附带请求头
```
Content-Type: application/json;charset=utf-8
Accept: application/json
Content-Length: {CONTENT_LENGTH}
X-XSRF-Protected: 1
```

### 获取相机信息
- 使用GET `/osc/info` 获取设备信息，建议不要超过1次/秒
```
{
	"manufacturer": "Arashi Vision",
	"model": "Insta360 One2",
	"serialNumber": "IXE3619AYS76P6",
	"firmwareVersion": "v1.18.43",
	"supportUrl": "https://www.insta360.com/product/insta360-one2/",
	"endpoints": {
		"httpPort": 80,
		"httpUpdatesPort": 80
	},
	"gps": false,
	"gyro": true,
	"uptime": 48,
	"api": [
		"/osc/info",
		"/osc/state",
		"/osc/checkForUpdates",
		"/osc/commands/execute",
		"/osc/commands/status"
	],
	"apiLevel": [
		2
	],
    "_sensorModuleType": "4K",
	"_vendorVersion": "v1.1_build1"
}
```
`_sensorModuleType`模组类型目前仅ONER型号支持，类型包括`4K/4K_Selfie` `Dual_Fisheye` `Leica/Leica_Selfie`,`Selfie`表示镜头为自拍方向（与触摸屏在同一方向）
- 使用POST `/osc/state` 获取设备状态，建议不要超过1次/秒
`/osc/state` 返回示例
```
{
	"fingerprint": "FPR_52923",
	"state": {
		"_cardState": "pass",
		"batteryLevel": 1,
		"storageUri": "http://192.168.42.1:80/DCIM/Camera01/"
	}
}
```
 `state._cardState`：表示卡状态，可能的值是 `noCard`无卡、`pass`有卡且格式正确、`noSpace`有卡空间不足、`invalidFormat`有卡格式不正确、`writeProtect`有卡写保护、`otherError`未知错误
 `state.batteryLevel`: 电量，0-1对应0-100的电量
 `state.storageUri`: 相机内拍摄照片的存储路径

 ***开始拍照前必须分别获取一次`/osc/info` 、`/osc/state`***

- 使用POST `/osc/commands/execute` 执行 `camera.getOptions` 获取相机的特性
`camera.getOptions` 参数示例：
```json
{
  "name":"camera.getOptions",
  "parameters": {
      "optionNames": [
          "iso",
          "isoSupport",
    "shutterSpeed",
    "shutterSpeedSupport",
          "hdr",
          "hdrSupport",
          "totalSpace",
          "remainingSpace",
          "photoStitching",
          "photoStitchingSupport",
          "captureInterval",
          "captureIntervalSupport",
          "captureMode",
          "_videoType",
          "_videoTypeSupport",
          "_timelapseResolution",
          "_timelapseResolutionSupport",
          "_timelapseInterval",
          "_timelapseIntervalSupport",
          "exposureProgram",
          "exposureDelay",
          "exposureDelaySupport",
          "_topBottomCorrection",
          "whiteBalance",
          "whiteBalanceSupport",
          "_dateTime",
          "_MuteEnable",
          "_batteryCapacity",
          "_sysTimestamp"
      ]
  }
}
```
`camera.getOptions` 正确返回示例：
```json
{
    "name": "camera.getOptions",
    "state": "done",
    "results": {
        "options": {
            "iso": 100,
            "isoSupport": [
                0,
                100,
                125,
                160,
                200,
                250,
                320,
                400,
                500,
                640,
                800,
                1000,
                1250,
                1600,
                2000,
                2500,
                3200
            ],
            "shutterSpeed": 0.001000,
            "shutterSpeedSupport": [
                0,
                120,
                100,
                80,
                60,
                50,
                40,
                30,
                15,
                8,
                2,
                1,
                0.500000,
                0.200000,
                0.100000,
                0.066660,
                0.033330,
                0.020000,
                0.016660,
                0.010000,
                0.008330,
                0.002000,
                0.001000,
                0.000500,
                0.000250,
                0.000125
            ],
            "hdr": "off",
            "hdrSupport": [
                "off",
                "hdr"
            ],
            "totalSpace": 0,
            "remainingSpace": 0,
            "photoStitching": "none",
            "photoStitchingSupport": [
                "none",
                "ondevice"
            ],
            "captureInterval": 3000,
            "captureIntervalSupport": {
                "minInterval": 3,
                "maxInterval": 120
            },
            "captureMode": "image",
            "_videoType": "normal",
            "_videoTypeSupport": [
                "normal",
                "timelapse"
            ],
            "_timelapseResolution": "5.7K",
            "_timelapseResolutionSupport": [
                "5.7K",
                "8K"
            ],
            "_timelapseInterval": 4000,
            "_timelapseIntervalSupport": [
                200,
                500,
                1000,
                2000,
                4000,
                10000,
                30000,
                60000,
                120000
            ],
            "exposureProgram": 2,
            "exposureDelay": 0,
            "exposureDelaySupport": [
                0,
                3,
                5,
                10,
                15,
                0
            ],
            "_topBottomCorrection": {
                "_topBottomCorrection": 0
            },
            "whiteBalance": "auto",
            "whiteBalanceSupport": [
                "auto",
                "incandescent",
                "fluorescent",
                "daylight",
                "cloudy-daylight"
            ],
            "_dateTime": "2023:03:23 16:05:41",
            "_MuteEnable": {
                "_MuteEnable": 0
            },
            "_batteryCapacity": {
                "powerType": 1,
                "powerLevel": 100
            },
            "_sysTimestamp": 60427274
        }
    }
}
```
`camera.getOptions` 参数错误返回结果:
```
{
	"name": "camera.getOptions",
	"state": "error",
	"error": {
		"code": "invalidParameterName ",
		"message": "parameter is not supported."
	}
}
```
- 每次成功连接相机时，执行 `camera.getOptions` 判断相机是否支持机内拼接功能
```
{
  "name":"camera.getOptions",
  "parameters": {
      "optionNames": [
          "photoStitchingSupport",
          "photoStitching"
      ]
  }
}
```
如果相机支持机内拼接，则会正确返回如下结果，此时可根据您的业务需求选择是否使用此功能。如果**返回错误结果**或`photoStitchingSupport`返回只有`none`，则代表相机不支持机内拼接功能
```
{
  "results": {
      "options": {
           "photoStitchingSupport":["none","ondevice"],
           "photoStitching":"none"
      }
  }
}
```
**`iso`表示当前iso值
`isoSupport`列出所有支持的iso值
`shutterSpeed`
`shutterSpeedSupport`
`aperture`
`hdrSpport`列出hdr选项
`hdr`显示当前HDR设置值
`totalSpace`表示内存卡总容量/Byte
`remainingSpace`表示剩余存储容量/Byte
`photoStitchingSupport`列出机内拼接选项
`photoStitching`显示当前机内拼接设置值
`captureInterval`间隔拍照时间
`captureIntervalSupport`间隔拍照支持的时间范围
`captureMode`拍摄模式
`_videoType`视频子模式
`_videoTypeSupport`视频子模式列表
`_timelapseResolution`延时摄影分辨率
`_timelapseResolutionSupport`可用的延时摄影分辨率列表
`_timelapseInterval`延时摄影的间隔时间，单位为毫秒
`_timelapseIntervalSupport`可用延时摄影的时间列表
`exposureProgram`曝光模式，1代表手动Manual，2自动NormalProgram，4快门优先ShutterPriority，9ISO优先ISOPriority
`exposureDelay`曝光延迟时间，启动拍摄后延迟一定时间后再曝光
`exposureDelaySupport`列出支持的曝光延迟时间
`_topBottomCorrection`照片水平校正开关
`whiteBalance`当前白平衡的模式名称
`whiteBalanceSupport`列出支持的白平衡设置值
`_dateTime`时间格式 "YYYY:MM:DD HH:MM:SS"，例如："2022:05:06 17:56:15"
`_MuteEnable`静音设置，0为不静音，1为静音
`_batteryCapacity`"powerType": 0 充电状态，0表示未充电，1表示正在充电
"powerLevel": 100 电量百分比 100表示100%，0表示没电池
`_sysTimestamp`系统时间戳，单位为us**
***无需每次请求都附带所有项目，根据需要即可***

### 拍照
#### setOptions
- 使用`camera.setOptions` 设置参数

- 拍摄HDR照片（拍摄完成后会有三张双鱼眼图片，需要在app中进行HDR合成和全景拼接）
```
{
	"name": "camera.setOptions",
	"parameters": {
		"options": {
			"captureMode":"image",
			"hdr": "hdr",
			"photoStitching": "none" //如果相机不支持机内拼接功能，请不要添加此参数
		}
	}
}
```
- 拍摄***机内拼接*** HDR照片命令如下：（拍摄完成后会返回一张进行过HDR合成及拼接的全景图片）
```
{
	"name": "camera.setOptions",
	"parameters": {
		"options": {
			"captureMode":"image",
			"hdr": "hdr",
			"photoStitching": "ondevice"  //如果相机不支持机内拼接功能，请不要添加此参数
		}
	}
}
```
正确返回：
```
{
    “name”:"camera.setOptions",
    "state":"done"
}
```
参数错误返回:
```
{
    "error": {
        "code": "invalidParameterName",
        "message": "Parameter options contains unsupported option captureInterval."
    }
}
```

#### takePicture
- 使用 `camera.takePicture` 拍摄照片
```
{
    "name":"camera.takePicture"
}
```
拍摄成功正确：
```
{
    "name":"camera.takePicture",
    "state":"inProgress",
    "id":"001996",
    "progress":
    {
        "completion":0
    }
}
```
拍摄失败--相机未激活，遇到此报错时，***需要提示用户下载Insta360官方app激活*** :
```
{
	"name": "camera.takePicture",
	"state": "error",
	"error": {
		"code": "unactivated",
		"message": "Please activate your camera in insta360 official app."
	}
}
```
拍摄失败-状态错误，遇到此报错时，***相机可能进入了`Standby`模式或者视频模式，重新发送`setOption`即可激活***：
```
{
	"name": "camera.takePicture",
	"state": "error",
	"error": {
		"code": "disabledCommand",
		"message": "Currently camera is not working in image mode"
	}
}
```


#### 查询status
- 使用 `/osc/commands/status` 查询当前命令的执行进度
```
{
    "id": "001996"
}
```
返回 `id` 为 `001996` 的拍摄命令执行进度：
```
{
    "name":"camera.takePicture",
    "state":"inProgress",
    "id":"001996",
    "progress":
    {
        "completion":0.5
    }
}
```
***`completion 0.5` 表示进度为50%***

#### 获取地址并下载
- 重复查询 `/osc/commands/status` 直到 `state` 为 `done`
使用 `results` 结果中的 `fileUrl` 来下载文件
```
{
	"name": "camera.takePicture",
	"state": "done",
	"results": {
		"_fileGroup": ["http://192.168.42.1:80/DCIM/Camera01/IMG_20180101_051113_00_042.jpg"],
		"_localFileGroup": ["/DCIM/Camera01/IMG_20180101_051113_00_042.jpg"],
		"fileUrl": "http://192.168.42.1:80/DCIM/Camera01/IMG_20180101_051113_00_042.jpg"
	}
}
```

#### 删除照片(optional)

- 通过 `camera.delete` 删除文件
```
{
    "name":"camera.delete",
    "parameters": {
        "fileUrls": [
            "url1",
            "url2",
            "url3",
            ...
            "urln"
        ]
    }
}
```
正确返回：
```
{
	"name": "camera.delete",
	"results": {
		"fileUrls": []
	},
	"state": "done"
}
```
错误返回:
```
{
    "error": {
        "code": "invalidParameterValue",
        "message": "Parameter url3 doesn't exist."
    }
}
```

#### 获取照片文件列表
- 使用`camera.listFiles`列出照片列表
```
{
	"name": "camera.listFiles",
	"parameters": {
		"fileType": "image",
		"entryCount": 1,
		"maxThumbSize": 100
	}
}
```
```
{
	"name": "camera.listFiles",
	"state": "done",
	"results": {
		"entries": [{
			"name": "IMG_20180106_180200_00_006.jpg",
			"fileUrl": "http://192.168.42.1:80/DCIM/Camera01/IMG_20180106_180200_00_006.jpg",
			"_localFileUrl": "/DCIM/Camera01/IMG_20180106_180200_00_006.jpg",
			"size": 4494224,
			"width": 6080,
			"height": 3040,
			"dateTimeZone": "2018:01:06 18:02:14+08:00",
			"_thumbnailSize": 37312,
			"isProcessed": true,
			"previewUrl": "",
			"thumbnail": "/9j/2wCEAAI...2Q==\n"//...表示省略
		}],
		"totalEntries": 3
	}
}
```

### 录像
#### setOptions
- 设置`captureMode`为`video`模式
```·
{
	"name": "camera.setOptions",
	"parameters": {
		"options": {
			"captureMode":"video"
		}
	}
}
```
正确返回：
```
{
	"name": "camera.setOptions",
	"state": "done"
}
```

#### startCapture

- 使用`camera.startCapture`开始录制视频
```
{
    "name":"camera.startCapture"
}
```

#### stopCapture

- 使用`camera.stopCapture`停止录制
```
{
    "name":"camera.stopCapture"
}
```
正确返回结果如下，其中包含`fileUrls`可以用来下载视频：
```
{
	"name": "camera.stopCapture",
	"state": "done",
	"results": {
		"fileUrls": [
			"http://192.168.42.1:80/DCIM/Camera01/VID_20180106_172302_10_005.mp4",
			"http://192.168.42.1:80/DCIM/Camera01/VID_20180106_172302_00_005.mp4"
		],
		"_localFileUrls": [
			"/DCIM/Camera01/VID_20180106_172302_10_005.mp4",
			"/DCIM/Camera01/VID_20180106_172302_00_005.mp4"
		]
	}
}
```

#### startTimelapse

1. timelapse拍照之前需要设置timelapse模式

```
{
    "name":"camera.setOptions",
    "parameters":{
        "options":{
            "captureMode":"video",
            "_videoType":"timelapse",
            "_timelapseInterval":1000
        }
    }
}
```

2. 开始拍摄

```
{
    "name":"camera.startCapture"
}
```

#### stopTimelapse

- 使用`camera.stopCapture`停止录制
```
{
    "name":"camera.stopCapture"
}
```
正确返回结果如下，其中包含`fileUrls`可以用来下载视频：
```
{
	"name": "camera.stopCapture",
	"state": "done",
	"results": {
		"fileUrls": [
			"http://192.168.42.1:80/DCIM/Camera01/VID_20180106_172302_10_005.mp4",
			"http://192.168.42.1:80/DCIM/Camera01/VID_20180106_172302_00_005.mp4"
		],
		"_localFileUrls": [
			"/DCIM/Camera01/VID_20180106_172302_10_005.mp4",
			"/DCIM/Camera01/VID_20180106_172302_00_005.mp4"
		]
	}
}
```

