# pixelman
A program for managing and distributing UE4 pixel streaming signaling servers and distributed rendering instances, supporting real-time start and stop of UE4 rendering instances, supporting distributed deployment of rendering instances, and unified scheduling and allocation by the manage module.

<div align="center">

[中文](README.ZH-CN.md) | [English](README.md)
</div>


## Features
- [x] Support multiple rendering machines, unified scheduling by the management end
- [x] Multiple rendering machines automatically allocate rendering instances according to the load situation
- [x] The management end can monitor the load situation of the rendering machine
- [x] The rendering service is automatically started and stopped, and the instance is automatically recycled after the user disconnects
- [x] Support multiple platforms (Windows, Linux, macOS)
- [x] Refactor the signaling server to support rendering instance allocation
- [ ] Support running rendering instances with Docker
- [ ] Support Kubernetes
- [ ] Support user authentication interface
- [ ] Support sending reset command to the client after the user disconnects, without the need to recycle the instance

## Install
1. First install Node.js, you can download and install the program from [https://nodejs.org](https://nodejs.org/zh-cn/download)
2. After installation, execute the command `npm install pixelman -g` in the console to install pixelman.
## Starting the service
The program is divided into management end and rendering end, which is a one-to-many relationship. When multiple rendering ends are connected to the management end, the management end will automatically allocate rendering instances according to the load situation of each rendering end. The management end is started by the command `pixelman manage`, and the rendering end is started by the command `pixelman render`.
The startup method is as follows:
1. First execute the command `pixelman genConfig genConfig manage` manage to generate the management end configuration file in the working directory.

```json
{
  "enableFrontend": true,  // Whether to enable the frontend program
	"enableDashboard": true,  // Enable dashboard or not. If enabled, it can be accessed via http://localhost/dashboard/.
  "httpHost": "127.0.0.1",  // Http server IP address
  "httpPort": 80,      // Http server port
  "streamerPort": 8888,  // Signaling server port
	"playerTimeout": 60,  // Client disconnection timeout. After this time, the instance will be recycled. (Unit: seconds)
  "renderServerHost": "127.0.0.1",  // Rendering server IP address
  "renderServerPort": 8866,   // Rendering server port
  "logLevel": "error",  // Log level, supports error, warn, info, verbose, debug, silly
  "logToFile": true,  // Write logs to file
  "logfilePath": "log",  // Log file directory
  "peerConnectionOptions": {}  // WebRtc configuration information for Unreal Engine and client handshake
}
```
2. Modify the configuration file and then execute the command `pixelman manage`  to start the signaling server.

3. Then execute `pixelman genConfig render` on the rendering machine to generate a rendering end configuration file in the working directory. (The installation service step also needs to be executed to install the pixelman program)
```json
{
  "renderName": "",     // Rendering end name
  "maxInstance": 1,  // Maximum number of instances running on the current node
  "renderServerHost": "127.0.0.1",   // Management server IP address
  "renderServerPort": 8866,  // Management server port
  "renderServerURL": null,  // Management server websocket link, this parameter is preferred. If set, renderServerHost and renderServerPort are invalid.
  "nvidiaSmiPath": "/usr/bin/nvidia-smi",  // The location of the NVIDIA graphics card SMI program, used to monitor the load situation of the rendering machine
  "task": {
    "cwd": "/home/kelin/LinuxClient2",  // The directory where the Unreal instance is running
    "exec": "sh ./MetaDemo.sh -RenderOffscreen -PixelStreamingURL=\"ws://127.0.0.1:8888/?taskId={taskId}\"" // The command to run the Unreal instance, {taskId} must be filled in, otherwise the instance cannot be effectively recycled
  },
  "logLevel": "error",  // Log level, supports error, warn, info, verbose, debug, silly
  "logToFile": true,  // Write logs to file
  "logfilePath": "log",  // Log file directory
  }
```
4. After modifying the configuration file, execute the `pixelman render` command to start the rendering service.

# contribution
