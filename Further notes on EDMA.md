# Further notes on EDMA
EDMA is heavily used in DPUs. It is essential to understand the basic usage of EDMA. In this document, I will combine the [Enhanced Direct Memory Access (EDMA3) Controller](https://www.ti.com/lit/ug/sprugs5b/sprugs5b.pdf) and test code located in mmwave_sdk_<ver>\packages\ti\drivers\edma\test to explain the code configuration of EDMA.
  
The diagram for one channel controller (CC) in EDMA is as follow:
  >![图片](https://user-images.githubusercontent.com/85469000/170001762-4a1806a3-8c07-4411-9318-e8d19fa14911.png)

  
## Code explaination
  
Like every C code, the test code start with *main()*, here initialize the SOC and start the *Test_task()*:  
  >![图片](https://user-images.githubusercontent.com/85469000/169942669-75f543ab-0314-4042-b749-79c8dc3da9b2.png)
  >![图片](https://user-images.githubusercontent.com/85469000/169942704-d58b1938-7d5f-4dba-942d-10327c1d3e31.png)

### Test_task()
  
In *Test_task()*, the code get the total number of EDMA instances on the device through *EDMA_getNumInstances()*. Lunch the *Test_instance()* on each instance.  
  >![图片](https://user-images.githubusercontent.com/85469000/169942883-02f0afef-3249-43c2-8aea-4b82eb3ecca9.png)
  
On IMR6843AOP, the *numInstances* is 2. a *EDMAinstance* refer to a channel controller.  
  >![图片](https://user-images.githubusercontent.com/85469000/169943496-c7c730d2-4de5-4348-8a99-d471cea411a5.png)


### Test_instance()
#### Initialization
In *Test_instance()*, the code first initialize and open the EDMA instance thorugh *EDMA_init()* and *EDMA_open()* at line 2888 and line 2891.  
  >![图片](https://user-images.githubusercontent.com/85469000/169943028-41c3e23f-343c-4218-beca-d7654e9fca84.png)  
  
#### Error monitoring settings
The *errorconfig* has following data fields:
  >![图片](https://user-images.githubusercontent.com/85469000/170005733-c9beca5b-01b0-4e9a-a529-2bb74179a28a.png)
  
The error monitoring function of CC and TC can be configurated. 
  >![图片](https://user-images.githubusercontent.com/85469000/170003114-5447f8a1-221c-40fd-a445-6888cd7dc8ed.png)
  
*isConfigAllEventQueues* is set to true means the setting apply to all queues. Otherwise, the idx of the queue should be specified in *eventQueueId*.  
Similarly, *isConfigAllTransferControllers* and *isEnableAllTransferControllerErrors* are both set to true, otherwise, *transferControllerId* and *transferControllerErrorConfig* need to configured.  
*isEventQueueThresholdingEnabled* is enabled. From my understanding, if the elements in the queues exceeds *eventQueueThreshold*, an event will be triggered to prevent overflow.
  
Configure the error call back function for CC and TC. *instanceInfo.isErrorInterruptConnected* and *instanceInfo.isTransferControllerErrorInterruptConnected[tc]* tells whether the interrupt from CC and TC is rounted to interrupt controller. This information is initialized by *EDMA_open()*.  
  >![图片](https://user-images.githubusercontent.com/85469000/170009969-8ada1e03-0c00-4585-b412-5ca6b25159b4.png)  
  
  If routed to interrupt cotroller, the call back function is linked.  
  >![图片](https://user-images.githubusercontent.com/85469000/170010130-a55a3767-917b-4fda-accc-6b511afe6e10.png)

  Call *EDMA_configErrorMonitoring()* to configurate the EDMA.
  >![图片](https://user-images.githubusercontent.com/85469000/170010908-93a04acd-2898-4bc6-8c16-43b1650d8049.png)

  #### Testing simultaneous unlinked unchained transfers
Call *Test_simultaneousUnchainedUnlinkedTransfersSuite()* to start the testing. This testing is *simultaneous unlinked unchained transfers*:  
  >![图片](https://user-images.githubusercontent.com/85469000/170011064-76a3e80e-4be4-4dd0-8e1a-8fce0a084108.png)
  
*Test_simultaneousUnchainedUnlinkedTransfersSuite()*:
  >![图片](https://user-images.githubusercontent.com/85469000/170011372-3aa0d8c9-b48b-448b-b471-3a5ad237b63f.png)

The testing function call *Test_init_testChannelConfig()* to initialize the EDMA channel:
  >![图片](https://user-images.githubusercontent.com/85469000/170011548-7160e968-5921-469b-a10f-4ba281b54d30.png)
  
The *Test_init_testChannelConfig()* then collect the *transferCompletionCallbackFxn* and *transferCompletionCode* of each channel. It also initialize the source, destination memory and *testState*, which record the state on the testing, including *isTransferDone* = false, *isTransferCompletionCodeInvalid* = false, *channelStart*, *channelEnd*, etc. There is no EDMA setting in this function, thus its code is neglected.
  
After the test is initalized, the test code call *Test_unchainedUnlinked*:
  >![图片](https://user-images.githubusercontent.com/85469000/170042968-1d95662c-02fd-4ddd-81fc-263ce0b972f6.png)
  
*Test_unchainedUnlinked()* first configure all test channels, set *transferCompletionCallbackFxnArg* to the EDMA handle:
  >![图片](https://user-images.githubusercontent.com/85469000/170044830-fe5262f2-0a98-4607-bde2-dd866c700c97.png)

There are intotal 4 channels in IWR6843AOP. From my understanding, each channel refer to one transfer controller. For each channel, the line 989 code get the configuration from *testChannelConfig[ ]*, which is an arguement get from upper layer, its variable name is *testChannelConfig__A_SINGLE_XFER_MIX_QDMA_DMA[ ]*. The *config* contain following data fields:
  >![图片](https://user-images.githubusercontent.com/85469000/170049938-e6e283b0-56b1-4f3d-8d14-2fb6e0542193.png)

  • Line 319 to 322 cofigurate *channelId*, *paramId* and *eventQueueId*. They are responsible for chossing the TC, PaRAM and queue. The *channelType* can choose between *EDMA3_CHANNEL_TYPE_DMA* or *EDMA3_CHANNEL_TYPE_QDMA*.
  >![图片](https://user-images.githubusercontent.com/85469000/170050298-7ed067c7-2eaa-480a-a607-6f244000098c.png)

  • Line 323 to 347 configurate the *paramSetConfig*.  
  • Line 323 and 324 configurate the source and destination addresses.  
  • Line 325 to 327 configure the Acnt, Bcnt and Ccnt.  
  • Line 328 set the reload value for Bcnt when Bcnt is decreased to 0, this is only revelent in A-type transfer (introduced later).  
  • Line 329 to 332 set the source and destination Bidx and Cidx.  
  • Line 333 control the linking of parameter set, which means when the transfer of this parameter set is completed, the parameter set with idx set by this value will be loaded into this parameter set.  
  >![图片](https://user-images.githubusercontent.com/85469000/170052978-8c24e6c2-ae48-45e9-93e9-f568d919212d.png)
  
  • Line 334 control the transfer type. There are 2 types of transfer in TI EDMA: A-type and AB-type. For A type, each event/transfer only transfer one dimension of data, which means one event only transfer Acnt bytes. Thus, Bcnt x Ccnt events are needed to transfer all datas. For AB type, one event/transfer transfer two dimensions of data, which means one event transfer Acnt x Bcnt bytes, which is entire row of data. Thus, Ccnt events are needed to transfer all datas. Since one EDMA transfer may need multiple event/transfer, the last one is called final TR (tranfer request) and all others are called intermediate TR.  
  >![图片](https://user-images.githubusercontent.com/85469000/170054530-74bc9e8c-9f7f-45ae-9ec9-6cb1dcd06f93.png)
  
  • Line 335 select the completion code, which is responsible for chaining. For example, consider an channel m need to chain to a channel n, which means the channel n will be triggered by m. Then, the idx of channel n need to be feed into the *transferCompletionCode* of channel m. Then, the channel m will trigger channel n when intermediate TR or final TR is submitted or completed, depending on the settings below.  
  • Line 336 and 337 set the addressing mode between linear and constant addressing mode.  
  • Line 342 set whether the parameter set is *static*. If this is set to true, it means this parameter set will be the last parameter set, no update or linking will be made. Otherwise this should be set to false.  
  • Line 343 select wether *early completion* is enabled. Early completion refer to that a transfer is considered completed once it is submitted to TC, although the TC may still doing the transfer.  
  • Line 344 to 347 set whether chaining or inturrpt are triggered after intermdiate TR and final TR. If *isFinalTransferInterruptEnabled* is enabled, the last TR will trigger the chaining; if *isIntermediateTransferInterruptEnabled* is enabled, the intermediate TRs will trigger the chaining. One example of chaining settings is as follow. The TCCHEN an ITCCHEN refer to whether the chaining is enabled for final and intermediate TR respectively.   
  >![图片](https://user-images.githubusercontent.com/85469000/170059492-e33a5698-e334-4bc4-a59e-e00182d37cbb.png)  
  
  Lastly are the callback function and a *specialAPI* settings:
  >![图片](https://user-images.githubusercontent.com/85469000/170060153-1635cee5-b5d2-424f-8e98-4d717de102a4.png)

#### Start the testing
Back to *Test_unchainedUnlinked()*, since the *isUseSpecialStartAPI* in config is set to false, the code call *EDMA_startTransfer* to start the transfer. The main difference of the special APIs are that the *EDMA_startDmaTransfer()* and *EDMA_startQdmaTransfer()* are faster if the type of transfer is known.
  >![图片](https://user-images.githubusercontent.com/85469000/170060407-f9fcac80-0400-41d2-8449-7250a7f67a33.png)

Then, the code compute the time needed to start the transfer API to finish entrie EDMA transfer. Since the intermediate interrupt is enabled, the code need to kick off each transfer event manually.  
  >![图片](https://user-images.githubusercontent.com/85469000/170062392-c7b4c6eb-a349-49a4-80cb-29bb25ffc97e.png)
  
Check whether a transfer is completed through *EDMA_isTransferComplete()*, if completed, the last arguement *testState.channelState[channel].isTransferDone* will be set to true.  
  >![图片](https://user-images.githubusercontent.com/85469000/170063051-7dcae7e6-0cab-4f22-bd25-0278982dfae8.png)

If a transfer is finished, kick start another transfer:
  >![图片](https://user-images.githubusercontent.com/85469000/170063501-c3c791a9-5b04-429f-b807-1e4155bfa03a.png)

Check whether the transfers are finished. Use *isTransfersDone* and *isChannelTransferDone* to indicate whether the current transfer is finished and whether all transfers are finished respectively.
  >![图片](https://user-images.githubusercontent.com/85469000/170063703-5bd3d7c2-089d-4993-ab4a-91c1a06198fe.png)
  
Check the result of the testings:
  >![图片](https://user-images.githubusercontent.com/85469000/170064215-6b237520-b193-43a3-b596-b6a0530493d6.png)

Clean up with *EDMA_disableChannel*:
  >![图片](https://user-images.githubusercontent.com/85469000/170064404-be951a31-e0f1-403a-a5d8-be6997f039fd.png)

### Test chained transfers
  The test is stared from *Test_instance()* here:
  >![图片](https://user-images.githubusercontent.com/85469000/170172264-2e638c22-9d05-4686-8991-4c6e40140f08.png)
  
  Similar to *Test_simultaneousUnchainedUnlinkedTransfersSuite()* mentioned previously, *Test_chainedTransfersSuite()* intialize the test records and call *Test_chainedTransfers()* to start the test:
  >![图片](https://user-images.githubusercontent.com/85469000/170172414-5f0f2da2-28d7-4fb8-a223-cbe2c7db7424.png)
  
  The use the configuration from *testChannelConfig__A_SINGLE_XFER_CHAIN* to set the EDMA channel. The difference between this configuration and the previous *testChannelConfig__A_SINGLE_XFER_MIX_QDMA_DMA* is that it disable the intermediate and final interrupt but enable the final chaining. Since the intermediate interrupt is disabled, there is no need to trigger each TR manually.
  >![图片](https://user-images.githubusercontent.com/85469000/170173752-2980ae8d-bc3b-47d7-96e9-621170099f5d.png)
  
  Set up the chaining of EDMA channels. In total there are 4 EDMA channels in the config, the *channelEnd* is 3. Thus the code loop through the first 3 EDMA channels and call *EDMA_chainChannels()* to set the chaining. The last EDMA channel should has only final transfer interrupt enabled, it should no chain to other EDMA channels.
  >![图片](https://user-images.githubusercontent.com/85469000/170179487-36949361-a0d9-427a-ab14-0dc75c1ea3aa.png)
  
  Next, the code kick off the first EDMA channel, then it will finish all the TRs can trigger next EDMA channel:
  >![图片](https://user-images.githubusercontent.com/85469000/170179802-53c07487-8676-4c63-8649-6ac3926e54af.png)

  All it left is wait for the completion and check the result:
  >![图片](https://user-images.githubusercontent.com/85469000/170179891-7b0336fe-b8fd-4404-931e-b2fee57768f1.png)

 
  




