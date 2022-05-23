# Radar Hardware Accelerator
In this document, I will introduce the programming of the radar, combining the test code provided by the TI and [Radar Hardware Accelerator](https://www.ti.com/lit/swru526). The source code is located at mmwave_sdk_<ver>\packages\ti\drivers\hwa\test\common. You can load the prebuilt binary file at mmwave_sdk_<ver>\packages\ti\drivers\hwa\test\xwr68xx\mss through CCS debug mode. [Video tutorial](https://training.ti.com/hardware-setup-mmwaveicboost-and-antenna-module)  
  
## Code explaination
All the code below are from mmwave_sdk_<ver>\packages\ti\drivers\hwa\test\common\main.c  
  
Like every C code, the test code start at main():
  >main()
  ![图片](https://user-images.githubusercontent.com/85469000/169736204-3380766f-d491-4bab-a326-5a0b25136f92.png)
    
The main() initialize the SOC and start a task to execute Test_initTask(). This is like starting a new threat to execute this function.  
  > Test_initTask()  
  ![图片](https://user-images.githubusercontent.com/85469000/169736399-e1bef63f-5c23-487e-98da-1fd1ba2bd02a.png)
  
In Test_initTask(), the first step is to initialize the HWA through HWA_init(). 
  >![图片](https://user-images.githubusercontent.com/85469000/169736556-a1c1f70c-5c2e-4966-947f-7a3a4db42d0e.png)
  
Then get a HWA handle from HWA_open(). This handle works like a reference to the opended HWA. All functions later that involve HWA need this handle to configure, treigger, shutdown, etc. the HWA. If the opening fail, the returned handle is NULL and the program need to be terminated.  
  
From line 281 to line 302, the code shut down and reopen the HWA in an intuitive way.  
  
  >![图片](https://user-images.githubusercontent.com/85469000/169737225-9244ea3e-f9d5-4d8b-b1da-a9a47e6e722d.png)
  
Now starts the actual part of the testing. The code first call the function configParamSetFFT(), which include configurating all the input formatter, output formatter, acceleration mode and so on. Let's look into it.
  >configParamSetFFT()
  >![图片](https://user-images.githubusercontent.com/85469000/169737125-5bf6ca79-187a-4225-a2c5-777c0a2e7e8d.png)
  >![图片](https://user-images.githubusercontent.com/85469000/169741326-edb53345-f06a-4467-b63c-5f361487617d.png)
  
  gHWATestParamConfig is an array of 16 HWA_ParamConfig. Used to configure the 16 parameter sets.  
  
  From line 876 to 879, the code configure the first parameter set, which is a dummy parameter set. It is triggered by software and trigger the next ping parameter.  
  
  Line 876 configure the trigger mode of the parameter set. There are 4 possible trigger modes:  
  • Immediate trigger (TRIGMODE = 000b): The state machine starts execution immediatly, does not wait for anything.  
  • Wait for processor-based software trigger (TRIGMODE = 001b):  This is used when the main processor needs to control the HWA directly. In this mode, the HWA monitor a bit in CR42ACCTRIG register. The bit is 0 by default, once it is set to 1 by the main processor, the HWA clear it and start execution.  
  • Wait for ADC buffer ping-to-pong or pong-to-ping switch (TRIG_MODE = 010b): In IWR6843AOP, the ADC buffered is shared with HWA local memory MEM0 and MEM1. ADC can be configured to switch from Ping-to-Pong or Pong-to-Ping. From my understanding, ADC can be switch from *writing to ADC buffer shared with MEM0 while HWA is reading from ADC buffer shared with MEM1* to *writing to ADC buffer shared with MEM1 and HWA is reading from ADC buffer shared with MEM0*. When this switching take place, it means the previos ADC buffer writing is completed, and HWA can start to read from the coresponding ADC buffer and start excution. Thus the HWA can be triggered by this switching. Noted that the HWA needs to ensure that the reading must be done before the next switching.  
  • Wait for the DMA-based trigger (TRIGMODE = 011b): In this mode, HWA is triggered when one of 16 DMA transfers related to HWA is finished. This DMA transfer is specified by DMA2ACC_CHANNEL_TRIGSRC register. When the DMA transfer is finished, the coresponding bit in DMA2ACCTRIG will be set, once this bit is set, the HWA is triggered.
  
