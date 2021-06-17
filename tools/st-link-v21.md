# Debugging and downloading via ST-LINK/V2-1

[^_^]:

[STM32 NUCLEO AND DFU USB BOOTLOADING](https://mobilewill.us/stm32-nucleo-and-dfu-usb-bootloading/)

[STM32F0, ST-link v2, OpenOCD 0.9.0: open failed](https://stackoverflow.com/questions/31638347/stm32f0-st-link-v2-openocd-0-9-0-open-failed)

*****************************

* ST-LINK/V2.1 is a development tool that can simulate and download ST chips online
* Support conversion to Link 0B emulator and USB serial port (slightly limited use)
* By brushing, you can transform into a DAP emulator and support non-ST chips
* Simultaneous support for emulation and USB to serial port, standard 2.54-10P 
interface; SWD, SWIM, serial port

First upgrade the firmware of the simulator, please refer to 
<https://www.st.com/en/development-tools/stsw-link007.html#documentation>.

Ӳ�������ӷ���������̲μ�
[ʹ�� ST-LINK/V2-1 ���������ص���·��](https://gitee.com/industry-ai/embedded-programming-windows/blob/master/readme.md#3-ʹ��-st-linkv2-1-���������ص���·��)

������ɺ��ϵ罫����õ� bin �ļ����ص����У������кܶ��֡�
�����ק�������ز��ɹ�������ճ���������ɹ������Դӹ������� STM32 ST-LINK Utility ��װ��ʹ�á�

![ST-LINK Utility](pix/utility.PNG)

��������μ�<https://www.cnblogs.com/pudonglin/p/14216141.html>


