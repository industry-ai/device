# Yabee 设备端

For English version please [switch to the English version](readme.md)

[^_^]:
https://github.com/industry-ai/device

[^_^]:
https://gitee.com/industry-ai/device

[^_^]:
F:\mygit\device

**********************************

Yabee 平板印花智能定位系统设备端资源站点

维护：胡继承 博士

## 1 板载固件

### 1.1 执行动作

板载固件最重要的是CAN帧的接收与解析。板载控制动作全部以函数的形式形成，并采用函数指针进行调用。

主循环通过检查动作链进行动作，一些时间戳动作由时钟将动作放入动作链。
为防止线程冲突将时间戳动作设置单独的动作链。动作链首尾相连形成环。
时钟动作可以由普通动作产生，也可以由中断产生。
实时动作和同步动作在中断中直接执行。

控制电路板连接到CAN网后会定期发送电路板上各devices的当前状态。

### 1.2 动作类型

* Real-time action: 直接在中断中执行，同步动作是一种实时动作
* Timeout action: 由时钟中断产生的action，可以是实时动作，也可以是普通动作
* Normal action: 通过主循环的动作链执行的action，由主循环依次执行

动作按编号进行管理，每个板载动作都有自己的编号，并在控制中心被按控制板进行维护，
即每个控制板都有自己的动作编号对照表被控制中心管理。

动作类型
```c
// 编号小于REALTIME_ACTION的动作为实时动作
#define	REALTIME_ACTION		0x01
// 编号小于TIMEOUT_ACTION的动作为时钟动作
#define	TIMEOUT_ACTION		0x02
// 编号小于NORMAL_ACTION的动作为普通动作
#define	NORMAL_ACTION		0x0F
```
动作按照函数指针的形式被保存在一个函数指针数组列表中，通过数组的下标对应动作编号，
从而可以迅速获得动作编号所对应的函数指针。


