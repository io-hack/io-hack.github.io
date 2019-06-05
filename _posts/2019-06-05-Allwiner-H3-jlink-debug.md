---
published: true
title: JLink调试全志H3(CortexA7四核)单核
---
## 下载
https://github.com/io-hack/io-hack.github.io/raw/master/dl/jlink-debug-AllwinerH3.tar.gz

## 使用方法
### 内核模块
- 针对Orangepi Lite
- Linux内核启动后，插入该内核模块，配置I/O为JTAG
- 退出该内核模块，恢复原来的I/O配置

### JLink脚本
- 依赖JLinkExe
- [SHELL] ./launch_jlink.sh 
- [SHELL] ./launch_jlink.sh 0
- [SHELL] ./launch_jlink.sh 1
- [SHELL] ./launch_jlink.sh 2
- [SHELL] ./launch_jlink.sh 3
- 默认连接core0
