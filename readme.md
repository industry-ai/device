# Device End of Yabee Tech

For Chinese version please go to [切换到中文版](readme_cn.md)

[^_^]:
https://github.com/industry-ai/device

[^_^]:
https://gitee.com/industry-ai/device

[^_^]:
F:\mygit\device

************************************

This is a repository of supplying resource for the development of Stencil Frame
Automatic Locating System for Flat Printing Pipeline, supported by Yabee tech Co.

Maintaining by: Dr. Jicheng Hu

For a review of the whole system, please see 
"[Overview of the Whole System](system/overview.md)".

## 1 On-board Firmware

Here we only talk about software associated part.
There are many ways to download and simulate debugging, and here in this project we chose 
the cheapest emulator ST-LINK/V2.1. For details please see 
[Debugging and downloading via ST-LINK/V2-1](tools/st-link-v21.md)

### 1.1 Executive actions

The most important aspect of on-board firmware is the receiving and parsing procedure
of the CAN frame, transported on the CAN network.
The on-board control actions are all formed in the form of executive functions and 
will be called using function pointers.

The main loop performs executive actions by checking the action chain, 
and some timestamp actions are put into the action chain by the clock. 
We set up a separate action chain for timestamp actions to prevent thread conflicts. 
The header and tailer of action chain are connected to form a ring.
Clock action can be generated by normal action or by interrupt.
Real-time and synchronized actions are performed directly during interrupts.

The current status of each device on the board is periodically sent whenever the control 
board is connected to the CAN network.

### 1.2 Action types

* Real-time action: Executed directly in an interrupt, a synchronous action is a real-time action
* Timeout action: The action resulting from time out of a clock interruption，
can be either real-time or normal
* Normal action: The action that is performed through the action chain of the main loop, 
which is executed in turn by the main loop

Actions are managed by index number, each on-board action has its own index, 
and is maintained by the control board in the control center, i.e. 
each control board has its own action index control table to be managed by the control center.

Action types:
```c
// action, with index number less than REALTIME_ACTION, is a realtime action
#define	REALTIME_ACTION		0x01
// action, with index number less than TIMEOUT_ACTION, is a timer action
#define	TIMEOUT_ACTION		0x02
// action, with index number less than NORMAL_ACTION, is a normal action
#define	NORMAL_ACTION		0x0F
```

Executive actions are saved in the form of function pointers in a list of function pointer arrays, 
which are marked with the function pointer array index, so that the function pointer corresponding to 
the array index can be obtained quickly.

### 1.3 Control protocol

Frame format to send out commands：

<table>
    <tr>
        <th rowspan="1">Board ID</th>
        <th colspan="8">data fields</th>
    </tr>
    <tr>
        <th rowspan="4">pre-defined</th>
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
            <td>Group ID</td>
            <td>Func code</td>
            <td>Action type</td>
            <td>action number</td>
            <td>data</td>
            <td>data</td>
            <td>data</td>
            <td>data</td>
        </tr>
        <tr>
            <td>pre-defined</td>
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

If the commands be received correctly, the bord of the device end will change the 
Func code to be 0xCB, and all other fields will be kept as original ones to send back.
The format is as follows:

<table>
    <tr>
        <th rowspan="1">Board ID</th>
        <th colspan="8">data fields</th>
    </tr>
    <tr>
        <th rowspan="4">pre-defined</th>
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
            <td>Group ID</td>
            <td>Func code</td>
            <td>Action type</td>
            <td>action number</td>
            <td>data</td>
            <td>data</td>
            <td>data</td>
            <td>data</td>
        </tr>
        <tr>
            <td>pre-defined</td>
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

The action types are currently include 3 types:
* real time action —— 0x00
* synchronous action —— 0x01
* normal action —— 0x02

### 1.4 An action example

This section starts with the simplest control of a single solenoid valve opening and closing, 
giving an example of a cycle test to control 8 sets of solenoid valves to open or to close.

The interface message response function `void CtestingDevicesDlg::OnBnClickedMfcbtn1 ( )` 
sends control action command to the solenoid valve numbered 1, opening or closing the solenoid 
valve. The status of the solenoid valve is sent by the heartbeat package that controls the 
circuit board. The current state of each devices on the board is periodically sent when the 
control board is connected to the CAN network.

First, add a control board object to the control center m_pMachine so that the control board 
device can interact with the control center to exchange information. Add an object to the function 
`void CMachineProofing::createModules ( )` that pushes the end control board 
`CBoardPushing`. `CBoardPushing:::Testing ( int )` is used to test the functionality of 
the board, testing a function by sending instructions to the board, which receives instructions 
and gives feedback.

The next step is to add sub-devices to `CBoardPushing`, which is done in the constructor.

