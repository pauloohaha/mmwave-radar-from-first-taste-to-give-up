# Radar Hardware Accelerator
In this document, I will introduce the programming of the radar, combining the test code provided by the TI and [Radar Hardware Accelerator](https://www.ti.com/lit/swru526). The source code is located at mmwave_sdk_<ver>\packages\ti\drivers\hwa\test\common. You can load the prebuilt binary file at mmwave_sdk_<ver>\packages\ti\drivers\hwa\test\xwr68xx\mss through CCS debug mode. [Video tutorial](https://training.ti.com/hardware-setup-mmwaveicboost-and-antenna-module)  
  
## Code explaination
All the code below are from mmwave_sdk_<ver>\packages\ti\drivers\hwa\test\common\main.c  
  
### main
Like every C code, the test code start at main():
  >main()
  ![图片](https://user-images.githubusercontent.com/85469000/169736204-3380766f-d491-4bab-a326-5a0b25136f92.png)
    
The main() initialize the SOC and start a task to execute Test_initTask(). This is like starting a new threat to execute this function.  
  > Test_initTask()  
  ![图片](https://user-images.githubusercontent.com/85469000/169736399-e1bef63f-5c23-487e-98da-1fd1ba2bd02a.png)
  
### Initialize and open the HWA
In Test_initTask(), the first step is to initialize the HWA through HWA_init(). 
  >![图片](https://user-images.githubusercontent.com/85469000/169736556-a1c1f70c-5c2e-4966-947f-7a3a4db42d0e.png)
  
Then get a HWA handle from HWA_open(). This handle works like a reference to the opended HWA. All functions later that involve HWA need this handle to configure, treigger, shutdown, etc. the HWA. If the opening fail, the returned handle is NULL and the program need to be terminated.  
  
From line 281 to line 302, the code shut down and reopen the HWA in an intuitive way.  
  
  >![图片](https://user-images.githubusercontent.com/85469000/169737225-9244ea3e-f9d5-4d8b-b1da-a9a47e6e722d.png)
  
### configure the HWA
Now starts the actual part of the testing. The code first call the function configParamSetFFT(), which include configurating all the input formatter, output formatter, acceleration mode and so on. Let's look into it.
  >configParamSetFFT()
  >![图片](https://user-images.githubusercontent.com/85469000/169737125-5bf6ca79-187a-4225-a2c5-777c0a2e7e8d.png)
  
  gHWATestParamConfig is an array of 16 HWA_ParamConfig. Used to configure the 16 parameter sets.  
  >![图片](https://user-images.githubusercontent.com/85469000/169741326-edb53345-f06a-4467-b63c-5f361487617d.png)
  
### Dummy parameter set
  From line 876 to 879, the code configure the first parameter set, which is a dummy parameter set. It is triggered by software and trigger the next ping parameter.  
  
  Line 876 configure the trigger mode of the parameter set. There are 4 possible trigger modes:  
  • Immediate trigger (TRIGMODE = 000b): The state machine starts execution immediatly, does not wait for anything.  
  • Wait for processor-based software trigger (TRIGMODE = 001b):  This is used when the main processor needs to control the HWA directly. In this mode, the HWA monitor a bit in CR42ACCTRIG register. The bit is 0 by default, once it is set to 1 by the main processor, the HWA clear it and start execution.  
  • Wait for ADC buffer ping-to-pong or pong-to-ping switch (TRIG_MODE = 010b): In IWR6843AOP, the ADC buffered is shared with HWA local memory MEM0 and MEM1. ADC can be configured to switch from Ping-to-Pong or Pong-to-Ping. From my understanding, ADC can be switch from *writing to ADC buffer shared with MEM0 while HWA is reading from ADC buffer shared with MEM1* to *writing to ADC buffer shared with MEM1 and HWA is reading from ADC buffer shared with MEM0*. When this switching take place, it means the previos ADC buffer writing is completed, and HWA can start to read from the coresponding ADC buffer and start excution. Thus the HWA can be triggered by this switching. Noted that the HWA needs to ensure that the reading must be done before the next switching.  
  • Wait for the DMA-based trigger (TRIGMODE = 011b): In this mode, HWA is triggered when one of 16 DMA transfer channels related to HWA is finished. This DMA transfer is specified by DMA2ACC_CHANNEL_TRIGSRC register. When the DMA transfer is finished, the coresponding bit in DMA2ACCTRIG will be set, once this bit is set, the HWA is triggered.  
    
In this test code, the trigger mode is selected to DMA-based trigger, and the channel is selected to HWA_TEST_SRC_TRIGGER_DMACH0. Later in main(), the HWA is triggered by manually setting the corresponding channel's bit:
  >![图片](https://user-images.githubusercontent.com/85469000/169743693-817b22f1-e8e1-448d-aa8e-909df8ff0d43.png)
  
### Ping parameter set
  From line 890 to 928, the code configure the Ping parameter set. Line 892 and 893 configure the trigger mode and acceleration mode of this parameter set:
  >![图片](https://user-images.githubusercontent.com/85469000/169744201-c4b433b1-922a-4030-a220-579437e628b2.png)
  
  The trigger mode is set to immediate. Thus, once the previous dummy parameter set is finished, this ping parameter set start execution right away. The acceleration mode is set to compute FFT.  
  
#### Input formatter
  Input formatter is used to access the datas in local MEMs, format and feed them into core computation unit. If can feed one complex sample per cycle. From line 894 to line 907, the code configure the input formatter of the HWA:

  Let's look at each setting in the test code:
  
  ![图片](https://user-images.githubusercontent.com/85469000/169749584-5e1b89f0-eef3-4f24-a356-d91624aae126.png)
  • Line 894 srcAddr: 0x0000 corresponds to the first memory location of ACCEL_MEM0 memory. srcAddr is defined in main() and passed to Test_initTask(), it is defined as a absolut address, thus need to subtract the SOC_HWA_MEM0 to get the absolut address w.r.t MEM0.
  >![图片](https://user-images.githubusercontent.com/85469000/169749369-9cff0c4c-3a98-4950-87f2-4a681250b5db.png)
  
  • Line 895 to 898 Acnt, Aidx, Bcnt, Bidx: Define the access pattern of the samples. Similar but not identical to the [EDMA access control](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/blob/Datapath/README.md#enhanced-dma-edma). Here, the sample size is defined by other parameters. A and B are two dimension of the data array, in which A can be understood as row and B can be understood as column. Acnt define how many elements are there in a row and Aidx define how many bytes seperate 2 consequtive elements in a row. Bcnt define the number of rows and Bidx define the seperation between the starting address of two rows.
  >![图片](https://user-images.githubusercontent.com/85469000/169753751-9e077eb2-f718-4783-8ec5-35d4ecec9a06.png)
  > Some numbers in this figure is not identical to the numbers in the test code, but the data pattern is same and should be helpful for understanding
  
  In the test code, there are HWA_TEST_NUM_RX_ANT = 4 antennas and each antenna has HWA_TEST_NUM_SAMPLES = 225 samples. The input data is arranged in interleaved mode, which means the datas for one RX antenna are located in one column. For examples, the 225 samples for RX0 are stored in array[0][0], array[1][0], array[2][0] ... array[224][0], the 225 samples for RX1 are stored in array[0][1], array[1][1], array[2][1] ... array[224][1]. Since HWA need to read datas for one RX antenna consequtively. Thus we need to access one column consequtively.
  
  >![图片](https://user-images.githubusercontent.com/85469000/169749664-1f631f33-9584-46ba-87ef-2aa810cffaff.png)
    
  Acnt is set to HWA_TEST_NUM_SAMPLES - 1, indicate the number of samples for one RX antenna.  
  AIdx is set to HWA_TEST_NUM_RX_ANT * HWA_TEST_COMPLEX_16BIT_SIZE, define the length of a row, which is the seperation between two consequtive data for one RX antenna.
  Bcnt is set to HWA_TEST_NUM_RX_ANT - 1, indicate the number of RX antennas.
  BIdx is set to HWA_TEST_COMPLEX_16BIT_SIZE, which is seperation between array[0][0] (the starting address of RX Ant 0), array[0][1] (the starting address of RX Ant 1), array[0][2] and array[0][3].
  
  >![图片](https://user-images.githubusercontent.com/85469000/169752313-579b55ae-7947-40e7-8772-653922a62ced.png)
  
  • Line 901 to 905 define the formatting of the data. In this test code, the input datas are signed complex 16-bit (for each I and Q) data. Thus srcRealComplex is set to 0 (0 for complex, 1 for real), srcWidth is set to 0 (o for 16-bit 1 for 32-bit), srcSign is set to 1 (1 for signed, 0 for unsigned), srcConjugate is set to 0 (0 for disabled).
  
  Since the ADC sample data is either 16-bit or 32-bit (for each I and Q) wide, but the input data to HWA can only be 24-bit wide. Thus we need to paddle some zero for 16-bit data and drop some bits for 32-bit data. When input is 16-bit wide, the srcScale specif how many bits are added at MSB. Here, srcScale is set to 8 means the data are extended to 24-bit by paddeling 8 bits at MSB.
  >![图片](https://user-images.githubusercontent.com/85469000/169753010-24e9e5e6-410b-4db5-9f6c-0c9b8cc17f4e.png)

  >![图片](https://user-images.githubusercontent.com/85469000/169753431-3b08f934-68bc-4443-a300-3351cba73ad3.png)
  >![图片](https://user-images.githubusercontent.com/85469000/169753478-9e9ea77d-79e5-4713-8b61-26954b906575.png)
  
  • Line 899, 900, 906 and 907 are not relevant in this test code, all are set to 0.

#### Output formatter
  >![图片](https://user-images.githubusercontent.com/85469000/169755768-6c7fa145-5c64-460f-844c-8feed3280fda.png)
  
  Similar to input formatter, the output formatter also can write datas in flexible pattern. dstAddr, dstAcnt, dstIdx, dstBIdx are similar to src, **dstBcnt is common to srcBcnt**.
  
  Othough the output of core computation unit is 24-bit wide, it can be cut or expand to 16-bit or 32-bit by setting dstWidth to 1 or 0 respectively. The cutting or expanding is similar to src. For 16-bit output, dstScale bits will be dropped at LSB.
  >![图片](https://user-images.githubusercontent.com/85469000/169756388-26fb0a37-8ef4-4601-82a3-2c3a2b5e6418.png)
  
  At dstAcnt does not need to be same with srcAcnt. It can be smaller than srcAcnt, thus some of the outputs at the end will be dropped. If some outputs at the beginning need to be dropped, you can set the dstSkipInit. The total number of output is (DSTACNT + 1) – REG_DST_SKIP_INIT.

#### Core computation unit
  
  • Line 918 select the function of HWA to be FFT  
  ![图片](https://user-images.githubusercontent.com/85469000/169761643-55fc6b61-11f7-4aae-8b99-6016cbc8cd11.png)
  
  • Line 922 to 925 set the windowing function of FFT.  
  >![图片](https://user-images.githubusercontent.com/85469000/169761763-5ba5914c-f799-4d5b-87b7-0092a4c30ca6.png)  
  
  When enabled, the HWA will multiple the input samples with Acnt number of real coefficients in a dedicated Window RAM.In window RAM, there are 1024 18-bit, signed, 2's-complement coefficient, the windowStart specify from which coefficient will the current iteration multiply. If the coefficient is symmetirc, winSymm can be enabled, thus only half of the coefficients need to be stored in the window RAM.
  >![图片](https://user-images.githubusercontent.com/85469000/169762239-0c6662d8-aff8-47ce-88a1-5fc29e9e60a9.png)

  • Line 919 to 921 specify some FFT settings.  
  >![图片](https://user-images.githubusercontent.com/85469000/169763396-d098e810-a63e-4f8f-89e1-4d349c683cfd.png)
  
  The FFT size must be a power of 2, such as 2, 4, 8, 16 .. 512 and 1024 are supported. fftSize is set to the log(2) of the FFT size. If the Acnt is less than the FFT size, the HWA will automatically paddel zeros after input. E.g. srcAcnt = 99, then fftsize is set to 7 (FFT size = 2^7 = 128), input formatter automatically apends 28 (128 - 99) zeros.  
  
  Other settings are less importent, you can refer to [Radar Hardware Accelerator](https://www.ti.com/lit/swru526) for more details.
  
  • Line 930 configurate the HWA.
  >![图片](https://user-images.githubusercontent.com/85469000/169826212-6237c36d-6bdb-4d21-90c5-59f3119c70f3.png)

### Dummy parameter set for Pong parameter set
  Similar to Ping dummy parameter set. But the trigger channel is changed from HWA_TEST_SRC_TRIGGER_DMACH0 to HWA_TEST_SRC_TRIGGER_DMACH1.
  ![图片](https://user-images.githubusercontent.com/85469000/169826492-52394f05-a06e-45b3-b0c7-64ef37d048d7.png)

### Pong parameter set
  The pong parameter set is almost identical to ping parameter set. For this test code, only the destination address is different. dstAddr is SOC_HWA_MEM2.
  ![图片](https://user-images.githubusercontent.com/85469000/169826747-ce559985-fd16-49ad-91bb-46a8fd50245b.png)

### Window coefficient fonfiguration
  • Line 325
  >![图片](https://user-images.githubusercontent.com/85469000/169827447-e97f9665-0799-469c-bdd4-b87e04ee99d9.png)  
  
  The prototype of the HWA_configRam() is:  
  >![图片](https://user-images.githubusercontent.com/85469000/169827614-eaa3200d-2436-4acb-825e-31e8d60820c9.png)
  
  The window coefficient *win_data225* is defined in *fft_window.h*:  
  >![图片](https://user-images.githubusercontent.com/85469000/169827852-52182995-a52b-42d3-9892-56e9c72beae2.png)

