# MSS and DSS
![image](https://user-images.githubusercontent.com/85469000/185585951-b59366de-c696-4efd-a8f9-81172b50a595.png)
![image](https://user-images.githubusercontent.com/85469000/185586022-b577aed7-808a-476f-8d31-fef1c8f939f6.png)

In this document, I will introduce the inilialization process of the DPC in distributed mode.  

## Initialization of the SOC and drivers
Firstly, the MSS and DSS will lauch *MmwDemo_initTask* and *MmwDemo_dssInitTask* independently to start their own initialization. These 2 function will firstly initialize the EDMA, HWA, GPIO and so on. *DPM_init* is for initializing the data structures. Then they will call *DPM_init* to wait for each other. Next, MSS will start the *CLI_task* and *mmwDemo_mssDPMTask* threads, other threads such as *MmwDemo_mmWaveCtrlTask* are not critical.  

*CLI_task* will first accpt all the CLI commands and load them into *gCLIMMWaveOpenCfg* or store the result through *MmwDemo_CfgUpdate* or *MMWave_addProfile* and so on. After the *sensorStart* in issued, the thread start to do the configurations. *MmwDemo_CLISensorStart* will call *MmwDemo_openSensor*, *MmwDemo_configSensor* and *MmwDemo_startSensor*. *MmwDemo_openSensor* is only for some initialization. *MmwDemo_configSensor* will call *MmwDemo_dataPathConfig* to start config the data path. *MmwDemo_dataPathConfig* will call *MmwDemo_DPM_ioctl_blocking* to send the configuration command into the DPM pipeline and block the thread before DSS finishe the configuration.
