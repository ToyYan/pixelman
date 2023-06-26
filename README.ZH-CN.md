# pixelman
虚幻引擎(unreal engine)像素流信令服务器和分布式管理的的程序，支持实时启停unreal engine渲染实例，支持渲染实例分布式部署，由manage模块统一调度分配。

<div align="center">

[中文](README.ZH-CN.md) | [English](README.md)
</div>


## 功能
- [x] 支持多台渲染机器统，由管理端统一调度
- [x] 多台渲染机器根据负载情况自动分配渲染实例
- [x] 管理端可以监控渲染机负载情况
- [x] 渲染服务自动启停，用户断开连接后自动回收实例
- [x] 支持多平台（windows，linux, macos）
- [x] 重构信令服务器,支持渲染实例分配
- [ ] 支持dacker运行渲染实例
- [ ] 支持k8s
- [ ] 支持用户认证接口
- [ ] 支持用户断开连接后发送重置命令给客户端，无需回收实例


## 安装
1. 首先安装nodejs，可以到 [https://nodejs.org](https://nodejs.org/zh-cn/download) 下载安装程序
2. 安装好后再控制台执行命令`npm install pixelman -g`安装pixelman
## 启动服务
程序分为管理端和渲染端，管理端和渲染端是一对多的关系，当有多个渲染端连接到管理端的时候，管理端会根据各个渲染端的负载情况自动分配渲染实例，管理端通过命令`pixelman manage`命令启动，渲染端通过`pixelman render`命令启动
启动方式如下：
1. 首先执行命令`pixelman genConfig genConfig manage` 会在工作目录下生成管理端配置文件
```json
{
	"enableFrontend": true,  // 是否启用前端程序
	"enableDashboard": true,  // 是否启用dashboard 启用后可以通过 http://localhost/dashboard/访问。
	"httpHost": "127.0.0.1",  // http服务器ip地址
	"httpPort": 80,      // http服务器端口
	"streamerPort": 8888,  // 信令服务器端口
	"playerTimeout": 60,  //  客户端断开超时时间，超过这个时间后，实例会被回收（单位 秒）
	"renderServerHost": "127.0.0.1",  // 渲染服务器IP地址
	"renderServerPort": 8866,   // 渲染服务器端口
	"logLevel": "error",  // 日志等级 支持 error, warn, info, verbose, debug, silly
	"logToFile": true,  // 日志写入到文件
	"logfilePath": "log",  // 日志文件目录
	"peerConnectionOptions": {}  // 虚幻引擎和客户端握手webRtc配置信息
}
```
1. 修改配置文件后 执行命令 `pixelman manage` 启动信令服务器。

2. 然后到渲染机器上执行`pixelman genConfig render`在工作目录下生成一个渲染端配置文件。（同样需要执行安装服务步骤，安装pixelman程序）
```json
{
	"renderName": "",     // 节点名称
	"maxInstance": 1,  // 当前节点运行最大实例数量
	"renderServerHost": "127.0.0.1",   // 管理服务器ip地址
	"renderServerPort": 8866,  // 管理服务器端口
	"renderServerURL": null,  // 管理服务器websocket链接，优先使用这个参数，如果设置后，renderServerHost和renderServerPort则无效。
	"nvidiaSmiPath": "/usr/bin/nvidia-smi",  // 英伟达显卡smi程序位置，用于监控渲染机器负载情况
	"task": {
		"cwd": "/home/kelin/LinuxClient2",  // 运行虚幻实例所在的目录
		"exec": "sh ./MetaDemo.sh -RenderOffscreen -PixelStreamingURL=\"ws://127.0.0.1:8888/?taskId={taskId}\"" // 运行虚幻实例需要执行的命令{taskId}必须填写，否则无法有效回收实例
	},
	"logLevel": "error",  // 日志等级 支持 error, warn, info, verbose, debug, silly
	"logToFile": true,  // 日志写入到文件
	"logfilePath": "log",  // 日志文件目录
	"peerConnectionOptions": {}  // 虚幻引擎和客户端握手webRtc配置信息
}
```
1. 修改好配置文件后执行 `pixelman render`命令启动渲染服务
