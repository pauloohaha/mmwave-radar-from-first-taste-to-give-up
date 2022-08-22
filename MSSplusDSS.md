# MSS and DSS
![image](https://user-images.githubusercontent.com/85469000/185585951-b59366de-c696-4efd-a8f9-81172b50a595.png)
![image](https://user-images.githubusercontent.com/85469000/185586022-b577aed7-808a-476f-8d31-fef1c8f939f6.png)

In this document, I will introduce the inilialization process of the DPC in distributed mode. The document is from *xwr68xx mmw Demo (for xWR68xx ISK board)* in SDK document.    

## Initialization of the SOC and drivers
Firstly, the MSS and DSS will lauch *MmwDemo_initTask* and *MmwDemo_dssInitTask* independently to start their own initialization. These 2 function will firstly initialize the EDMA, HWA, GPIO and so on. *DPM_init* is for initializing the data structures. Then they will call *DPM_init* to wait for each other. Next, MSS will start the *CLI_task* and *mmwDemo_mssDPMTask* threads, other threads such as *MmwDemo_mmWaveCtrlTask* are not critical.  

*CLI_task* will first accpt all the CLI commands and load them into *gCLIMMWaveOpenCfg* or store the result through *MmwDemo_CfgUpdate* or *MMWave_addProfile* and so on.  
After the *sensorStart* is issued, the thread start to do the configurations. *MmwDemo_CLISensorStart* will call *MmwDemo_openSensor*, *MmwDemo_configSensor* and *MmwDemo_startSensor*.  
*MmwDemo_openSensor* is only for some initialization.  

## Datapath configuration
*MmwDemo_configSensor* will call *MmwDemo_dataPathConfig* to start config the data path. *MmwDemo_dataPathConfig* is responsible for configurating ADC buffer and the datapath. The function first set the common configurations such as number of subframes and compensation settings, then it set each subframe one by one. It set the ADC buffer through *MmwDemo_ADCBufConfig*, set the datapath through *DPM_ioctl* or *MmwDemo_DPM_ioctl_blocking*, which is a *DPM_ioctl* plus a *Semaphore_pend*, to send the configuration message command into the DPM pipeline. This message contain a command and a configuration, the command specify on which domain it should be executed and what exactly is the configuration about.  
The *xxx_dpmTask* on both domains will call *DPM_execute* continuously, which will call *DPM_msgRecv* to receive the message from the DPM pipeline. *DPM_msgRecv* will dequeue a message from DPM message pipeline and execute it. If this message is a command for the other domain, the message will be relayed to the other domain through *DPM_msgPostProcess*.  
If the configuration message command find the right domain, it will be processed by *DPC_ObjectDetection_ioctl* from *objectdetection.c* or *objectrangehwa.c*. The ioctl function will call the coresponding configuration function according to the command.  
For HWA pre-start config, the command is *DPC_OBJDETRANGEHWA_IOCTL__STATIC_PRE_START_CFG* and the function is *DPC_ObjDetRangeHwa_preStartConfig*, which then call *DPC_ObjDetRangeHwa_rangeConfig*->*DPU_RangeProcHWA_config* to configurate the HWA.  
For DSP pre-start config, the command is *DPC_OBJDET_IOCTL__STATIC_PRE_START_CFG* and the function is *DPC_ObjDetDSP_preStartConfig*, which then call the coresponding functions to configurate the DSP.  

## Open sensor
After the datapath is configurated, *MmwDemo_startSensor* is called, which start the data path and the sensor. Start the data path means call *MmwDemo_dataPathStart* -> *DPM_start* to send start message into DPM message pipeline. When *DPM_execute* receive the message, it call *DPM_msgStartHandler* -> *DPC_ObjectDetection_start* to trigger the HWA (MSS) to set it ready for ADC buffer data, and report. Start the sensor means call *MMWave_start* to let the RF start sending chirps according to the settings applied earlier.
