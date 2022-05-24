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
Call *Test_simultaneousUnchainedUnlinkedTransfersSuite()* to start the testing.  
  >![图片](https://user-images.githubusercontent.com/85469000/170011064-76a3e80e-4be4-4dd0-8e1a-8fce0a084108.png)
  
*Test_simultaneousUnchainedUnlinkedTransfersSuite()*:
  >![图片](https://user-images.githubusercontent.com/85469000/170011372-3aa0d8c9-b48b-448b-b471-3a5ad237b63f.png)

The testing function call *Test_init_testChannelConfig()* to initialize the EDMA channel:
  >![图片](https://user-images.githubusercontent.com/85469000/170011548-7160e968-5921-469b-a10f-4ba281b54d30.png)
  
The *Test_init_testChannelConfig()* then collect the *transferCompletionCallbackFxn* and *transferCompletionCode* of each channel. It also initialize the source, destination memory and *testState*, which record the state on the testing, including *isTransferDone* = false, *isTransferCompletionCodeInvalid* = false, *channelStart*, *channelEnd*, etc. There is no EDMA setting in this function, thus its code is neglected.
  
After the test is initalized, the test code call *Test_unchainedUnlinked*:
  >![图片](https://user-images.githubusercontent.com/85469000/170042968-1d95662c-02fd-4ddd-81fc-263ce0b972f6.png)
  
*Test_unchainedUnlinked()* first configure all test channels:
  >![图片](https://user-images.githubusercontent.com/85469000/170044830-fe5262f2-0a98-4607-bde2-dd866c700c97.png)

There are intotal 4 channels in IWR6843AOP. From my understanding, each channel refer to one transfer controller. For each channel, the line 989 code get the configuration from *testChannelConfig[ ]*, which is an arguement get from upper layer, its variable name is *testChannelConfig__A_SINGLE_XFER_MIX_QDMA_DMA[ ]*. One of the channel setting is shown bellow:
  >![图片](https://user-images.githubusercontent.com/85469000/170021279-2f3a7349-a342-445a-9a9f-aaf8a6290650.png)


