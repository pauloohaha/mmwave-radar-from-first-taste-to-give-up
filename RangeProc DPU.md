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
  
  Then, the ADC bit, numChirpsPerChirpEvent and ADCBufData is set. *adcDataIn* here is a global variable. It will be filled with sample data in from test file.
  >![图片](https://user-images.githubusercontent.com/85469000/170202594-aa6a6359-644b-46c8-9ed8-5300836d14dd.png)
  
  >![图片](https://user-images.githubusercontent.com/85469000/170203166-d3b62333-082c-4509-9882-5e80ddfa9a47.png)

  The *radarCube* is also linked. This is also a global variable.
  >![图片](https://user-images.githubusercontent.com/85469000/177093568-86f098d7-e058-4198-a3d1-7c9114b7d709.png)
 
  Back to *rangeProcDpuTest_Task()*, the program then read *numDataReadIn& and calculate the total number of tests. The *numDataReadIn* is 15 and *numTests* is 504.
  >![图片](https://user-images.githubusercontent.com/85469000/177095763-9644ad27-9240-4ab1-a973-64b7c9879b82.png)
  
  Next, the program loop through each test. It first read the *numTxAntennas*, *numRangeBins* and *numChirpsPerFrameRef*.
  >![图片](https://user-images.githubusercontent.com/85469000/177095845-9d21451f-c4a2-469e-acda-2a297538b06f.png)
  
  Next, check whether the setting is valid:
  >![图片](https://user-images.githubusercontent.com/85469000/177096222-8f2315e8-20a6-4e72-ad1c-aaf96d9a4b3e.png)
  
  Print out the setting:
  >![图片](https://user-images.githubusercontent.com/85469000/177096400-4650fe43-c3ff-414d-8941-cd0005814ffc.png)
  >![图片](https://user-images.githubusercontent.com/85469000/177096424-eeb9ab0b-623e-43e4-a6b0-62536dea8b88.png)
  
  Next, the program set some settings depent on different tests, such as num of RX, TX ANT, num of ADCsamples and so on. They are recorded in *testConfig*. And the test setting is printed:
  >![图片](https://user-images.githubusercontent.com/85469000/177098065-b515c02c-a04a-4ae0-b020-f677a31b29ec.png)
  >![图片](https://user-images.githubusercontent.com/85469000/177098084-fed941a6-4543-4418-ab9a-e1435cc61eb6.png)
  
  Then, the *Test_setProfile()* is called:
  >![图片](https://user-images.githubusercontent.com/85469000/177098227-e0be10df-04a2-411a-b050-17553cede51c.png)
  
  In *Test_setProfile()*, the program first copy the settings recorded in *testConfig* to *param*.
  >![图片](https://user-images.githubusercontent.com/85469000/177100954-ba9df73f-3864-49d8-b74e-be92ac14c014.png)
  
  Some setting about butterfly stage:
  >![图片](https://user-images.githubusercontent.com/85469000/177101150-1139258a-bfd3-4ec8-b3fb-2b00bb08f9d8.png)
  
  The *rxChanOffset* is set. In non-interleaved mode, this is used to indicate where the data for this RX anteena start in ADC buffer.
  >![图片](https://user-images.githubusercontent.com/85469000/177101506-712c20d6-2782-4f4a-9746-e9e1b3d52d26.png)

  EDMA for data input and output is set:
  >![图片](https://user-images.githubusercontent.com/85469000/177102061-74c0d953-1ad6-449a-ada6-ab4e0c2aa6de.png)
  >![图片](https://user-images.githubusercontent.com/85469000/177102297-245cdeb0-289a-4191-b99a-d11397b6d2b4.png)
  
  The *radarCube* settings:
  >![图片](https://user-images.githubusercontent.com/85469000/177102388-2fc33fb9-3e65-43ae-84f7-69efa9099ed1.png)

  Back to *rangeProcDpuTest_Task()*, the program then call *DPU_RangeProcHWA_config()*. It uses the *rangeProcDpuCfg* to initialize *rangeProcDpuHandle*, which is acutally of *rangeProcHWAObj* type. The handle it self is a pointer of viod type.  
  >![图片](https://user-images.githubusercontent.com/85469000/177103028-512c6cb9-3ea6-434e-8174-e770553c407b.png)
  
  The *DPU_RangeProcHWA_config()* first perform a series of checking on the settings. Such as whether the handles are valid, the data format is supported, windowing size is correct, radar cube size is correct and so on.  
  >![图片](https://user-images.githubusercontent.com/85469000/177104271-44674035-7ba0-4074-a61b-54210bb542e6.png)
  
  After the checking, the program copy the *calibDcRangeSigCfg* from *pConfigIn* to *rangeProcObj* and call *rangeProcObj()*, which basically copy every needed data from *pConfigIn* to *rangeProcObj*.  
  >![图片](https://user-images.githubusercontent.com/85469000/177104738-2db2c615-5072-41fd-b36e-b4ebf38913e2.png)
  
  Copy *param*:  
  >![图片](https://user-images.githubusercontent.com/85469000/177105586-fcd7e8e5-0fed-4bd4-bca2-1e8f7ebac341.png)
  
  Copy data buffers' pointer and  the *rxChanOffset* if in non-interleaved mode.  
  >![图片](https://user-images.githubusercontent.com/85469000/177105651-a064a472-229b-41ae-bfae-454f6f41fd90.png)

  Set the data format and return error if two cases that can not be handled happen:
  >![图片](https://user-images.githubusercontent.com/85469000/177106106-9ac8036b-4eef-43ea-8a9c-460b63622b5b.png)

  Here the arrangement of 4 HWA parameter sets. After the data in EDMA is finished, it will write dataInTrigger[0] and dataInTrigger[1] into HWA's DMA2ACCTRIG to trigger these 2 parameter sets to trigger the processing of HWA. When the data out EDMA is finished, it will trigger the dummy parameter set of HWA.
  >![图片](https://user-images.githubusercontent.com/85469000/177106163-37d6e2a2-7c3b-4c39-9478-46b0711e86a8.png)
  >![图片](https://user-images.githubusercontent.com/85469000/177121169-60e7be89-a925-4bf8-bfb3-f10894dea34f.png)

  Inside *rangeProcHWAObj*, other than *edmaDoneSemaHandle*, *hwaDoneSemaHandle*, *dcRangeSigCalibCntr*, *calibDcRangeSigCfg*, *hwaMemBankAddr*, *calibDcNumLog2AvgChirps*, *inProgress*, *numProcess* and *numEdmaDataOutCnt*, all others are set.
  
  DC calibration settings are set next:
  >![图片](https://user-images.githubusercontent.com/85469000/177107513-2222ea7a-35c8-420d-8c6f-96d7263e200f.png)
  
  In *rangeProcHWA_dcRangeSignatureCompensation_init()*, *dcRangeSigMean*, *dcRangeSigMean*, *dcRangeSigCalibCntr* and *calibDcNumLog2AvgChirps* are set.
  
  After the initialization of *rangeProcHWAObj*, the HWA is reset and window parameters are set:  
  >![图片](https://user-images.githubusercontent.com/85469000/177108053-539f9a32-b301-4bbb-a667-e9a1b11f7fe1.png)

  At last, all the preperation is finished and *rangeProcHWA_HardwareConfig()* is called to acutally config the EDMAs and HWA.  
  Depend on whether the ADC sample is interleaved or not, the function call the subsequent function corespondingly:  
  >![图片](https://user-images.githubusercontent.com/85469000/177122216-1e2cce5d-dd90-4dee-8761-5d14786a2aea.png)
  
  Take non interleaved *rangeProcHWA_ConifgNonInterleaveMode()* as example. It mainly config the EDMA for data in (if isolated mode), the HWA parameter set and EDMA for data out.  
  
  The *rangeProcHWA_ConfigEDMA_DataIn()* is used to configure data in EDMA. It uses data path EDMA (DPEDMA) untilities to set the EDMA easier. For noninterleaved, it uses *DPEDMA_configSyncAB()* to condigure EDMA. Its argument is listed below. The *chanCfg* is set ealier, thus use directly. The chainCfg control the chaning of the dataIn EDMA. This EDMA is chained to data in one hot signature EDMA to trigger the HWA after the data is moved into the HWA local memory. Only the final chaining is needed. Since this is non interleaved mode, the data for one antenna is stored linearly. So the EDMA read the data sequentially and write to two HWA local memory M0 and M1 respectively.
  >![图片](https://user-images.githubusercontent.com/85469000/177124187-656e9334-9dd7-471e-bc41-0f164892be79.png)
  >![图片](https://user-images.githubusercontent.com/85469000/177124331-d6729e71-c8ba-47f6-94ca-6759c22c6027.png)

  Next, the data in one hot signature EDMA transfer is configurated through *DPEDMAHWA_configTwoHotSignature()*. It use the *&pHwConfig->edmaInCfg.dataInSignature* channel and trigger the two HWA parameter set by writing into DMA2ACCTRIG register in HWA. Here, the *dataInTrigger[0]* and *dataInTrigger[1]* are 6 and 8 repectively.  
  >![图片](https://user-images.githubusercontent.com/85469000/177125660-7dc3cd76-2056-4e04-aae9-1ad1dbd10562.png)
  
  The *rangeProcHWA_ConfigHWA()* is used to configure the HWA. It is responsible for configurating 4 parameter sets in HWA, one dummy set and one peocess parameter set for each PING and PONG. Take PING as example. In isolated mode, the data in EDMA trigger the process parameter set directly after the data is moved into the local memory of HWA. Thus, the parameter set index for the PING's process parameter set is *dataInTrigger[0]*  = 6. In mapped mode, the process parameter set is triggered when ADC buffer is switched from PING to PONG or PONG to PING. Dummy parameter sets are triggered by software through DMA trigger.
  >![图片](https://user-images.githubusercontent.com/85469000/177246098-399ad375-c20e-4426-8afa-d12e34b474f8.png)
  >![图片](https://user-images.githubusercontent.com/85469000/177246395-910e0eca-cc96-469c-9890-8cad99d68429.png)
  
  The configuration of dummy parameter set is easy. The trigger mode is DMA, DMA trigger source is set to its parameter set IDX, the accelerate mode is set to NONE.
  >![图片](https://user-images.githubusercontent.com/85469000/177246714-34195fa4-6dd7-4380-8280-8b8e7680f811.png)
  
  Next are the settings of PING process parameter set. Firstly, the trigger mode is configurated. In mapped mode, the process parameter is triggered when the ADC buffer switch from ping to pong or ping to ping, thus set to DEF. In isolated mode, it is triggered by the completion of data in EDMA, thus the trigger mode is DMA.
  >![图片](https://user-images.githubusercontent.com/85469000/177246814-6b4fa90a-fc52-4fcd-a422-3b38793a2da3.png)
  
  Next are the settings for data input and output formattor:
  >![图片](https://user-images.githubusercontent.com/85469000/177247048-e4163a2d-74b4-454d-86e6-84c09f7557c9.png)
  
  Here set the format of input data and output data. Each of them are either interleaved or not. For input data, the *DPIF_RXCHAN_INTERLEAVE_MODE* indicate whether the input data is interleaved. If interleaved, the data for one RX ANT is stored in one column. If not interleaved, the data for one RX ANT is in one row. For output to the HWA local memory and radar cube, the *rangeProc_dataLayout_RANGE_DOPPLER_TxAnt_RxAnt* indicate whether it is interleaved or not.  
  >![图片](https://user-images.githubusercontent.com/85469000/177247887-e7da8991-d068-43fd-8c07-3fa514c00cfa.png)  
  
  If the *rangeProc_dataLayout_RANGE_DOPPLER_TxAnt_RxAnt* is ture, the output data looks like:  
  >![图片](https://user-images.githubusercontent.com/85469000/177252643-282b828f-c51c-4b4e-b622-5c99bf5848b5.png)  
  
  If it is not ture, the output data looks like:  
  >![图片](https://user-images.githubusercontent.com/85469000/177252670-55e870b4-abbe-4f96-acdb-b9a4241ef87f.png)  

  
  Next, the finished interrupt is set. The *destChanPing* should be used latter in data out EDMA. *destChanPing* = 0, *destChanPong* = 1.
  >![图片](https://user-images.githubusercontent.com/85469000/177249056-3e3e2321-ff49-4a85-839a-0165ff3e5c9d.png)
  
  The dummy parameter set and process parameter set for PONG are similar, they are omited.
  
  Next, the data out EDMA is set. Data out EDMA is triggered by the completion of process parameter sets. It move the data from HWA local memory to radar cube. It also trigger the data out one hot signature EDMA.
  >![图片](https://user-images.githubusercontent.com/85469000/177249313-9de6e9be-4101-4ff3-b065-199ddd5eeedb.png)
  >![图片](https://user-images.githubusercontent.com/85469000/177249432-1083df12-977c-4ab7-81be-8050abdefa91.png)
  
  If the output data in HWA local memory is interleaved (*rangeProc_dataLayout_RANGE_DOPPLER_TxAnt_RxAnt* is true), the radar cube is also interleaved. In this case, the output data can be copied to radar cube directly. aCnt is length 4 RX ANT, bCnt is num of rangeBins, cCnt is the number of chirps per PING/PONG.  
  >![图片](https://user-images.githubusercontent.com/85469000/177255375-96ed4aa6-4866-4736-ac78-e7bd023d8c8a.png)
  >![图片](https://user-images.githubusercontent.com/85469000/177255426-e9239d1b-fe52-4b8b-896f-3648c20357a9.png)
  
  If the output data in HWA local memory is non-interleaved (*rangeProc_dataLayout_RANGE_DOPPLER_TxAnt_RxAnt* is false), the radar cube is also not interleaved. This case is complex. In the radar cube, results of one TX ANT is stored continuously. In each TX antenna datas, each doppler chirp is stored continuously. Each PING or PONG has 3 EDMA ParamSet linked together, moving the data of each TX ANT to their especific destination address. For PING as example, there are 3 shallow EDMA linked one by one, they move data from M2 to TX1, TX3 and TX2 repectively.
  >![图片](https://user-images.githubusercontent.com/85469000/177256710-a3cb29fd-cfe7-47e6-b354-55e1dea8f39e.png)
  
  Firstly, the destination of each TX, RX ANT combination is calculated. destAddr[0][] refer to PING, destAddr[1][] refer to PONG. destAddr[][0, 1, 2] refer to TX 1, 3, 2 repectively.
  >![图片](https://user-images.githubusercontent.com/85469000/177257171-805a9ac1-c97b-4ec8-8ecf-d2dc933050c0.png)

  Next, the code initialize three dummy EDMA ParamSet and lnk them together.
  >![图片](https://user-images.githubusercontent.com/85469000/177260640-865f0468-1546-43a6-8d3c-efbd19d2a208.png)
  >![图片](https://user-images.githubusercontent.com/85469000/177260772-204b9c87-f799-4c4b-8d2c-af1dd8be0a5b.png)\
  
  Then, each of them is loaded with correct sourceAddr and destAddr.
  >![图片](https://user-images.githubusercontent.com/85469000/177260832-af07f956-9dd8-41e7-b4c0-4ffe029a4105.png)






  




  





