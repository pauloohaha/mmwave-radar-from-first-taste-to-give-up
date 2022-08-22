# MSS and DSS
In this document, I will introduce the inilialization process of the DPC in distributed mode. The document is from *xwr68xx mmw Demo (for xWR68xx ISK board)* in SDK document.  

In this demo, the range proc is executed on HWA and rest of the processing are executed on DSP. This is controlled by *OBJDET_NO_RANGE* defined in mss and dss makefile.  

## Initialization of the SOC and drivers
![image](https://user-images.githubusercontent.com/85469000/185585951-b59366de-c696-4efd-a8f9-81172b50a595.png)
![image](https://user-images.githubusercontent.com/85469000/185586022-b577aed7-808a-476f-8d31-fef1c8f939f6.png)

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

## Data path execution
![image](https://user-images.githubusercontent.com/85469000/185851411-fcfde30a-a8a1-4c41-a18b-2ab8eaaaefab.png)

After the sensor is open, the BSS will start to send chirps. It will notice the MSS and DSS through interrupts. These interrupts are predefined:
![image](https://user-images.githubusercontent.com/85469000/185836474-e31304e6-5577-42b9-97be-1577cf5e40ce.png)  

During *DPM_init* -> *DPM_initDPC*, these interrupts are linked to *DPM_chirpISR* and *DPM_frameStartISR*  
![image](https://user-images.githubusercontent.com/85469000/185836587-0f2ba149-bdbe-4a26-b88e-b29835719a03.png)  

For MSS, it only has *DPC_ObjectDetection_frameStart*:  
![image](https://user-images.githubusercontent.com/85469000/185845384-4d34237b-d536-4285-b599-e0f57ef028b9.png)  

For DSS, it has *DPC_ObjectDetection_frameStart* and *DPC_ObjectDetection_chirpEvent* when DSP is responsible for range processing:    
![image](https://user-images.githubusercontent.com/85469000/185845485-4123f3f9-da16-47f4-9ecd-08208db08a9c.png)

For DSS, frame start is only for checking whether the previous frame has been completed and do some logging. But for MSS, frame start function is also responsible for invoking *DPM_notifyExecute()*, which will set *ptrDPM->executeDPC* to notify *DPM_execute()* to invok the DPC execute funtion:  
![image](https://user-images.githubusercontent.com/85469000/185846787-9378ebf3-c289-4f15-976a-28b2db9be49c.png)
![image](https://user-images.githubusercontent.com/85469000/185846913-92e2f2aa-7208-4e4a-b8cb-879bed7d427e.png)

In MSS, *DPC_ObjectDetection_execute()* is responsible for waiting for HWA to complete range FFT and send the result to DSS. *DPU_RangeProcHWA_process()* wait for the HWA through 2 semaphores:  
![image](https://user-images.githubusercontent.com/85469000/185851844-2db5008f-4ee6-4017-936d-9b0fe9e8c4c9.png)

After the range FFT is completed, *DPM_relayResult()* -> *ptrDPM->ptrDomainFxnTable->sendResult* -> *DPM_mboxSendResult()* is called to send result to DSS. It send a message into DPM message pipeline:  
![image](https://user-images.githubusercontent.com/85469000/185853683-39693156-3e74-42e9-a942-ececf8cd7a65.png)

The *DPM_execut()* at DSS receive the message and call *DPM_msgResultHandler()* to handle. It then call *ptrDPM->procChainCfg.injectDataFxn* -> *DPC_ObjectDetection_dataInjection* -> *DPM_notifyExecute* to notify execute.

