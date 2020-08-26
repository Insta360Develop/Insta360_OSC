# Insta360 OSC 使用文档
### 概览
**本文档基于[Google OSC](https://developers.google.com/streetview/open-spherical-camera)规范编写**
使用Insta360 ONE X、ONE R时，设备连接到相机WiFi热点，此时相机的IP地址：192.168.42.1
**强烈建议** ： 所有 `osc/commands` 命令收到回复之后再发送下一条

- 发送请求时请附带请求头
```
Content-Type: application/json;charset=utf-8
Accept: application/json
Content-Length: {CONTENT_LENGTH}
X-XSRF-Protected: 1
```

### 获取相机状态
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
	"_vendorVersion": "v1.1_build1"
}
```

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
```
{
  "name":"camera.getOptions",
  "parameters": {
      "optionNames": [
          "iso",
          "isoSupport",
          "hdrSupport",
          "hdr",
          "totalSpace",
          "remainingSpace",
          "photoStitchingSupport",
          "photoStitching"
      ]
  }
}
```
`camera.getOptions` 正确返回示例：
```
{
  "results": {
      "options": {
           "iso": 200,
           "isoSupport": [100, 200, 400, 800, 1600],
           "hdrSupport":[hdr, off],
           "hdr":"off",
           "totalSpace":"31906594816",
           "remainingSpace":"10597040128",
           "photoStitchingSupport":["none","ondevice"],
           "photoStitching":"none"
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
**`iso`表示当前iso值
`isoSupport`列出所有支持的iso值
`hdrSpport`列出hdr选项
`hdr`显示当前HDR设置值
`totalSpace`表示内存卡总容量/Byte
`remainingSpace`表示剩余存储容量/Byte
`photoStitchingSupport`列出机内拼接选项
`photoStitching`显示当前机内拼接设置值**
***无需每次请求都附带所有项目，根据需要即可***

### 拍照
#### setOptions
- 使用`camera.setOptions` 设置参数
`shots` 表示拍摄张数(目前只能是3张) `increment`表示每一张的EV间隔值，一般`increment`的值为2,表示拍摄的3张原片分别为-2ev,0ev,2ev。



- 拍摄HDR照片
```
{
	"name": "camera.setOptions",
	"parameters": {
		"options": {
			"hdr": "hdr",
			"photoStitching": "none"
		}
	}
}
```
- 拍摄***机内拼接*** HDR照片命令如下：
```
{
	"name": "camera.setOptions",
	"parameters": {
		"options": {
			"hdr": "hdr",
			"photoStitching": "ondevice"
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
- 使用 `/osc/cammands/status` 查询当前命令的执行进度
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
- 重复查询 `/osc/cammands/status` 直到 `state` 为 `done`
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
    "name":"camera.stopCapture"
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