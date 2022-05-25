# RangeProc DPU

The first step of the data processing is do the FFT on ADC samples for range calculation. Thus, Range Processng DPU is the first DPU data flow through. This DPU take RF data from ADC buffer as input, calculate the 1D FFT, and same the results in RadarCube. In this document, I will explain the test code located at *mmwave_sdk_<ver>\packages\ti\datapath\dpu\rangeproc\test* with the help of documents at *mmwave_sdk_03_05_00_04/packages/ti/datapath/dpu/rangeproc/docs/doxygen/html/index.html*.
  
  ### Reference
  1. [Further notes on EDMA](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/blob/Datapath/Further%20notes%20on%20EDMA.md#further-notes-on-edma)  
  2. [Radar Hardware Accelerator](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/blob/Datapath/Further%20notes%20on%20Radar%20Hardware%20Accelerator.md#radar-hardware-accelerator)  
  
## Code explaination
  The test code below are from *mmwave_sdk_<ver>\packages\ti\datapath\dpu\rangeproc\test\hwa_main.c*  
  The lib functions for dpu are located at *mmwave_sdk_<ver>\packages\ti\datapath\dpu\rangeproc\src\rangeprochwa.c*  
  Like every C code, the test code start with *main()*. It initialize the SOC and start a thread to execute *rangeProcDpuTest_Task()*.
  >![图片](https://user-images.githubusercontent.com/85469000/170195916-b020a0c3-9c65-430a-b8b4-2eabd6d9c51e.png)

  *rangeProcDpuTest_Task()* first call there functiomns: *rangeProcDpuTest_hwaInit()*, *rangeProcDpuTest_edmaInit()*, and *rangeProcDpuTest_dpuInit()* to initialize the required hardware and data:
  >![图片](https://user-images.githubusercontent.com/85469000/170195978-5e31d2b9-61d0-4966-a088-5ed403c9bab3.png)
  
  #### rangeProcDpuTest_hwaInit()
  As explained in [Radar Hardware Accelerator](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/blob/Datapath/Further%20notes%20on%20Radar%20Hardware%20Accelerator.md#radar-hardware-accelerator), the code call *HWA_init()* and *HWA_open()* to initialize the HWA and get a handle:
  >![图片](https://user-images.githubusercontent.com/85469000/170196211-c97fe61c-e5b0-45e9-aeee-3545ad16ca4e.png)
  
  #### rangeProcDpuTest_edmaInit()
  As explained in [Further notes on EDMA](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/blob/Datapath/Further%20notes%20on%20EDMA.md#further-notes-on-edma), the code get the number for EDMA intances, initialize, open and set the error monitoring functions.
  >![图片](https://user-images.githubusercontent.com/85469000/170196410-6b827e95-8047-468c-bc95-35a100553fd8.png)
  
  #### rangeProcDpuTest_dpuInit()
  The DPU init function call the lib function *DPU_RangeProcHWA_init()*. Since my aim is to creat my own DPU, let's look into this function. It's source code is located at *mmwave_sdk_<ver>\packages\ti\datapath\dpu\rangeproc\src\rangeprochwa.c*.
  >![图片](https://user-images.githubusercontent.com/85469000/170196845-a2d8c19e-1eb5-4c82-9a33-93fa441d8598.png)
  
  The function *DPU_RangeProcHWA_init()* mainly initialize the variable *rangeProcObj*, which store the DPU settings such as HWA MEM address, data in/out trigger, semaphore for hwa and edma done, etc. The, this variable is returned as *DPU_RangeProcHWA_Handle*.  
  >![图片](https://user-images.githubusercontent.com/85469000/170197952-5380c604-676f-464b-bafd-9aba6e8941f7.png)

  In this specific function, it initialize the variable and allocate memory for it. Then, it get the HWA bank memory address by *HWA_getHWAMemInfo()*, set the semaphores for HWA and EDMA.  
  >![图片](https://user-images.githubusercontent.com/85469000/170198569-0ab7d613-a7b1-443a-bc75-6666b91e0552.png)
  
  Back to the *hwa_main.c*, the returned handle is passed to *rangeProcDpuHandle*， which is a global variable.
  
  Thus far, the test code has finished the initialization. It then starts to configure the test through *rangeProcDpuTest_dpuCfg()*. The main purpose of this function is to configure a global variable *DPU_RangeProcHWA_Config rangeProcDpuCfg*.
  >![图片](https://user-images.githubusercontent.com/85469000/170201774-e67e927f-f136-43c7-9cc1-5baaff7a08a9.png)
  
  *rangeProcDpuCfg* has follow 3 data fields:
  >![图片](https://user-images.githubusercontent.com/85469000/170201406-b043d1c2-565b-4e6a-9589-3608b408da0c.png)
  
  The function first configure the *hwRes* data field. Including the starting parameter set Idx, the total number of parameter sets needed (which is 4, 2 for each Ping and Pong). The windowing settings and input settings. The edma settings are ignored here. I suppose it will be set later during interleaving settings.
  >![图片](https://user-images.githubusercontent.com/85469000/170201962-feec972f-bb4f-47c4-82e1-f9c19cf4ab1f.png)
  
  Then
  >![图片](https://user-images.githubusercontent.com/85469000/170202594-aa6a6359-644b-46c8-9ed8-5300836d14dd.png)
  
  >![图片](https://user-images.githubusercontent.com/85469000/170203166-d3b62333-082c-4509-9882-5e80ddfa9a47.png)


  



