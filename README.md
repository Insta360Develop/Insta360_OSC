# Insta360 OSC Get-Started

## [中文文档](README_zh.md)

### Overview
**Insta360 OSC is implemented based on [Google OSC](https://developers.google.com/streetview/open-spherical-camera)**
**Note** When your client connected to the AP of Insta360 ONE X, ONE X2, ONE R, ONE RS, X3, X4, X5, the ip address of the camera is：`192.168.42.1`
**Strongly recommended** ： Never send  `osc/commands` request before you get the response of the previous one

- HTTP Header required
```
Content-Type: application/json;charset=utf-8
Accept: application/json
Content-Length: {CONTENT_LENGTH}
X-XSRF-Protected: 1
```

### Get Camera Info
- Send a GET request to  `/osc/info` to get basic info of the camera，the frequency of the request is recommend to no more than 1 request per second.
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

`_sensorModuleType` :  Indicates the current lens module type of the camera, only supported on ONE  R. Available values are `4K`, `4K_Selfie`,`Dual_Fisheye`,`Leica`,`Leica_Selfie`. The `Selfie` suffix indicates the lens is facing the same direction with touch screen. 

- Send a POST request to  `/osc/state` to get state of the camera device.

  `/osc/state`  response example:
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

- `state._cardState`：card state of the camera device. Available values:

  + `noCard`: no card detected
  +  `pass`: A card detected with supported format
  +  `noSpace`: A card detected, but no space left
  +  `invalidFormat:`: A card detected, but the format is not supported
  +  `writeProtect`: A card detected, with WriteProtected on
  +  `otherError`: Other unknown error

   

- `state.batteryLevel`: battery level of the camera device, values in [0,1]
- `state.storageUri`: the storage path where photos are saved.

 ***Remember to send  request to `/osc/info` 、`/osc/state` before starting to take picture***

- Use `camera.getOptions` command to get options and features of the camera device. `photoStitchingSupport` and `photoStitching` are newly added options to tell if the camera device support ondevice-stitching.
`camera.getOptions` request example：
```
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
`camera.getOptions` response example

```
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
`camera.getOptions` response example (with error):

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
After successfully connected to the camera，execute  `camera.getOptions` to know if the camera firmware support on-device stitching.

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

If the on-device stitching is supported, the response will be like below. Otherwise, the response may contain error or `photoStitchingSupport` only has value `none`.

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

`iso`:  value of current iso
`isoSupport`: list of supported iso values
`hdrSpport`:  list of supported hdr values
`hdr`: current hdr value
`totalSpace`:  total SDcard space in byte
`remainingSpace`: remaining space in byte
`photoStitchingSupport`: list of supported photoStitching modes
`photoStitching`: current photoStitching mode
`captureInterval`Interval photo time
`captureIntervalSupport`Time Range Supported by Interval photo 
`captureMode`capture mode
`_videoType`video submode
`_videoTypeSupport`list of video submodes
`_timelapseResolution`timelapse resolution
`_timelapseResolutionSupport`List of available timelapse resolutions
`_timelapseInterval`timelapse interval in millisecond
`_timelapseIntervalSupport`list of times available for timeLapse
`exposureProgram`Exposure mode, 1 -> manual Manual, 2 -> automatic Normal Program, 4 -> shutter priority ShutterPriority, 9 -> ISO priority ISOPriority
`exposureDelay`Exposure delay time, after starting shooting, delay for a certain period of time before exposing
`exposureDelaySupport`List of supported exposure delay times
`_topBottomCorrection`Photo Level Correction Switch
`whiteBalance`The mode name of the current white balance
`whiteBalanceSupport`List supported white balance settings
`_dateTime`Time format "YYYY:MM:DD HH:MM:SS", for example: "2022:05:06 17:56:15"
`_MuteEnable`Mute setting, 0 is no mute, 1 is mute
`_batteryCapacity`"powerType": 0 charging status, 0 means not charging, 1 means charging "powerLevel": 100 battery percentage 100 means 100%, 0 means no battery
`_sysTimestamp`System timestamp in microseconds(us)

***SetOptions/GetOptions allows you to query/set options as need, just put required options in the optionsName array***

### TakePicture
#### setOptions
- use `camera.setOptions` command to set params

- set HDR mode (Three  original images will be returned upon `takePicture` successfully executed)
```
{
	"name": "camera.setOptions",
	"parameters": {
		"options": {
			"captureMode":"image",
			"hdr": "hdr",
			"photoStitching": "none" //add this option only when on-device stitching is supported
		}
	}
}
```
- set HDR mode with **ondevice-stitching** ：(One stitched image with HDR effect will be returned upon `takePicture` successully executed)
```
{
	"name": "camera.setOptions",
	"parameters": {
		"options": {
			"captureMode":"image",
			"hdr": "hdr",
			"photoStitching": "ondevice" //add this option only when on-device stitching is supported
		}
	}
}
```
response on success：
```
{
    “name”:"camera.setOptions",
    "state":"done"
}
```
response on error:
```
{
    "error": {
        "code": "invalidParameterName",
        "message": "Parameter options contains unsupported option captureInterval."
    }
}
```

#### takePicture
- use `camera.takePicture` command to start taking picture
```
{
    "name":"camera.takePicture"
}
```
response on success：
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
response on error --camera not activated，**Please  remind user to download Insta360 official App to activate their camera when get this error**:
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
response on error-state error，**This error often happens when camera is in  standby mode or  video mode, call setOptions (as described above) and try again**：
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


#### check capture status
- use `/osc/commands/status` to check capture status
```
{
    "id": "001996"
}
```
`id` identifies the capture task
response example:
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
***`completion 0.5` means the capture progress is 50% now***

#### Get file url and download 
- check `/osc/commands/status` repeatedly (recommend once per second ) until `state` is  `done` in the response
make a http request to download the file via `fileUrl` in the  `results` 
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

#### Delete the photo (optional)

- use `camera.delete` to delete the file
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
response on success：
```
{
	"name": "camera.delete",
	"results": {
		"fileUrls": []
	},
	"state": "done"
}
```
response on error:
```
{
    "error": {
        "code": "invalidParameterValue",
        "message": "Parameter url3 doesn't exist."
    }
}
```

#### Get File Lists
- use `camera.listFiles` to list files on the device
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

### Recording
#### setOptions
- set `captureMode` to `video` before start recording 
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
response on success：
```
{
	"name": "camera.setOptions",
	"state": "done"
}
```

#### startCapture

- use `camera.startCapture` to start recording
```
{
    "name":"camera.stopCapture"
}
```

#### stopCapture

- use `camera.stopCapture` to stop recording 
```
{
    "name":"camera.stopCapture"
}
```
Make a http request to download the video files via `fileUrls`：
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

1. Timelapse mode needs to be set before taking pictures

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

2. Start capture

```
{
    "name":"camera.startCapture"
}
```

#### stopTimelapse

- use `camera.stopCapture` to stop recording 
```
{
    "name":"camera.stopCapture"
}
```
Make a http request to download the video files via `fileUrls`：
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
