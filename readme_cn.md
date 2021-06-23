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

若需要对整个系统有一个粗略的了解，请阅读 
“[系统开发环境综述](system/overview_cn.md)”。

## 1 板载固件

这里我们仅仅讨论软件相关部分。下载及仿真调试的方法有很多，此项目我们选择最便宜的 ST-LINK/V2.1 
作为仿真器，参见 [Debugging and downloading via ST-LINK/V2-1](tools/st-link-v21.md)

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


### 1.3 控制协议

发送指令格式：

<table>
    <tr>
        <th rowspan="1">Board ID</th>
        <th colspan="8">数据域</th>
    </tr>
    <tr>
        <th rowspan="4">预设</th>
        <tr>
            <td>Byte0</td>
            <td>Byte1</td>
            <td>Byte2</td>
            <td>Byte3</td>
            <td>Byte4</td>
            <td>Byte5</td>
            <td>Byte6</td>
            <td>Byte7</td>
        </tr>
        <tr>
            <td>组号</td>
            <td>功能码</td>
            <td>动作类型</td>
            <td>动作号</td>
            <td>数据</td>
            <td>数据</td>
            <td>数据</td>
            <td>数据</td>
        </tr>
        <tr>
            <td>预设</td>
            <td>0xCA</td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
    </tr>
</table>

若指令被正确接收，功能码改变为 0xCB，其它报文由接受到指令的控制板原文返回。格式如下：

<table>
    <tr>
        <th rowspan="1">Board ID</th>
        <th colspan="8">数据域</th>
    </tr>
    <tr>
        <th rowspan="4">预设</th>
        <tr>
            <td>Byte0</td>
            <td>Byte1</td>
            <td>Byte2</td>
            <td>Byte3</td>
            <td>Byte4</td>
            <td>Byte5</td>
            <td>Byte6</td>
            <td>Byte7</td>
        </tr>
        <tr>
            <td>组号</td>
            <td>功能码</td>
            <td>动作类型</td>
            <td>动作号</td>
            <td>数据</td>
            <td>数据</td>
            <td>数据</td>
            <td>数据</td>
        </tr>
        <tr>
            <td>预设</td>
            <td>0xCB</td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
    </tr>
</table>

动作类型表：
* 实时动作 —— 0x00
* 时钟动作 —— 0x01
* 普通动作 —— 0x02


### 1.4 An action example

本节从一个最简单的控制单个电磁阀开闭入手，给出一个循环测试控制8组电磁阀开闭的例子。

界面消息响应 `void CtestingDevicesDlg::OnBnClickedMfcbtn1 ( )` 会向编号为 1 的电磁阀发送控制动作，
打开或关闭该电磁阀。电磁阀的状态由控制电路板的心跳包发送过来。
控制电路板连接到 CAN 网后会定期发送电路板上各 devices 的当前状态。

首先在控制中心向`m_pMachine`中增加一个控制电路板对象，这样该控制电路板设备才能与控制中心进行信息交互。
在函数`void CMachineProofing::createModules ( )`中增加一个推进端控制板`CBoardPushing`的对象。
`CBoardPushing::Testing ( int )`用于对该电路板的功能进行测试，测试某个功能即发送相应的指令给电路板，
电路板收到指令后进行反馈。

下一步给`CBoardPushing`增加子设备，这个在构造函数中完成。


