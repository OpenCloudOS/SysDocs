# 硬件兼容性测试工具
用于验证OS和硬件设备的兼容性
## 工具编译

`./build_rpm.sh`

制作好的rpm包位于build/RPMS目录下

## 工具安装

`rpm -ivh tst-hct-1.0.0-1.tl3.x86_64.rpm`

安装后工具文件默认放置在/var/lib/tst-hct目录下

## 测试执行

执行测试对环境有如下要求：

- 已配置好yum软件源，支持安装必须的软件包
- 如果要执行network相关测试，需要有另一台测试机辅助测试，本机可通过ssh秘钥登录辅助测试机的root用户
- 如果要执行storage相关测试，需要有专用于测试的存储设备，测试中该设备**数据会被清除**
- 对network、storage和gpu进行测试，需要通过参数指定待测设备，用法见帮助信息

### 列出工具测试项
命令`tst-hct list`

命令执行示例
```bash
bash# tst-hct list
TST_TS_TOPDIR=/var/lib/tst-hct

   1 : cpu.benchmark                  : ./testcase/cpu.benchmark.sh
   2 : cpu.calculate                  : ./testcase/cpu.calculate.c
   3 : cpu.hotplug                    : ./testcase/cpu.hotplug.sh
   4 : cpu.list                       : ./testcase/cpu.list.sh
   5 : cpu.power                      : ./testcase/cpu.power.sh
   6 : cpu.schedule                   : ./testcase/cpu.schedule.c
   7 : gpu.list                       : ./testcase/gpu.list.sh
   8 : gpu.vendor.nvidia              : ./testcase/gpu.vendor.nvidia.sh
   9 : memory.allocate                : ./testcase/memory.allocate.c
  10 : memory.hotplug                 : ./testcase/memory.hotplug.sh
  11 : memory.list                    : ./testcase/memory.list.sh
  12 : network.connect                : ./testcase/network.connect.sh
  13 : network.list                   : ./testcase/network.list.sh
  14 : network.tcp                    : ./testcase/network.tcp.sh
  15 : network.udp                    : ./testcase/network.udp.sh
  16 : storage.list                   : ./testcase/storage.list.sh
  17 : storage.mount                  : ./testcase/storage.mount.sh
  18 : system.backtrace               : ./testcase/system.backtrace.sh

total 18 testcase
```

### 执行测试
命令`tst-hct test`

- 网络设备测试要求：
  * 执行网络测试需要传入参数 --network 指定**本机网口**名称和**对端服务器IP**
  * 需要配置本机到对端服务器的**ssh免密登录**
- 存储设备测试要求：
  * 执行存储设备测试需要传入参数 --storage 指定可以用于测试的设备名
  * 改设备在测试前不能被挂载
- 显卡设备测试要求：
  * 显卡测试目前只支持NVIDIA显卡，测试时需要传入 --cuda 参数指定待测试的显卡编号(nvidia-smi列出显卡编号)
  * 测试环境需要安装对应驱动和cuda工具包：https://developer.nvidia.com/cuda-toolkit-archive

命令执行示例
```bash
bash# tst-hct test --cuda 0,1,2,3 --network eth0:192.168.0.1 --network eth0:192.168.1.1 --storage sdb,sdc,nvme1
Get system information...
Run test...
TST_TS_TOPDIR=/var/lib/tst-hct

            ts_setup                       ==> PASS
   1/18   : cpu.benchmark                  ==> PASS (cost 20.110)
   2/18   : cpu.calculate                  ==> PASS (cost 0.020)
   3/18   : cpu.hotplug                    ==> PASS (cost 19.220)
   4/18   : cpu.list                       ==> PASS (cost 0.130)
   5/18   : cpu.power                      ==> PASS (cost 23.300)
   6/18   : cpu.schedule                   ==> PASS (cost 0.810)
   7/18   : gpu.list                       ==> SKIP (cost 5.820)
   8/18   : gpu.vendor.nvidia              ==> PASS (cost 0.060)
   9/18   : memory.allocate                ==> PASS (cost 227.850)
  10/18   : memory.hotplug                 ==> PASS (cost 547.430)
  11/18   : memory.list                    ==> PASS (cost 0.230)
  12/18   : network.connect                ==> PASS (cost 0.070)
  13/18   : network.list                   ==> PASS (cost 0.130)
  14/18   : network.tcp                    ==> PASS (cost 0.060)
  15/18   : network.udp                    ==> PASS (cost 0.060)
  16/18   : storage.list                   ==> PASS (cost 1.430)
  17/18   : storage.mount                  ==> PASS (cost 0.080)
  18/18   : system.backtrace               ==> PASS (cost 0.590)
            ts_teardown                    ==> PASS

total: 18
 pass: 17
 skip: 1
 fail: 0

execute ./tsuite run fail, cost 847.620
```
