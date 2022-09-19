# Pick up raw ADC data
## Idea
Record the adc buffer data for the first chirp only.  

In total, we need to extract:  
1. ADC raw data  
2. ADC raw data length  
3. range FFT output  
4. range FFT output length  
from the range proc DPU.

For these datas, we need to declear them in *objectdetection.h* -> *DPC_ObjectDetection_ExecuteResult*, *rangeprocdsp_internal.h* -> *rangeProcDSPObj*, and *rangeprocdsp.h* -> *DPU_RangeProcDSP_HW_Resources*.  
Allocate memory to it at *objectdetection.c* -> *DPC_ObjDetDSP_preStartConfig*. Send its address into *DPC_ObjDetDSP_rangeConfig*，where it will be load into the *rangeCfg.hwRes*, which is the *pConfig* in *DPU_RangeProcDSP_config*.  
Then, the *pConfig* is send into *rangeProcDSP_ParseConfig*, where the address of the *ADCBufferDataRecord* can be stored into *rangeProcObj*. It will be passed to *DPU_RangeProcDSP_process* later, in which the adc data can be copied into the *ADCBufferDataRecord*.  
In *objectdetection.h* -> *DPC_ObjectDetection_execute()*, the four data will be load into the result buffer after the *DPU_RangeProcDSP_process*.


## Modification made
1.  
In *objectdetecion.h*, define the four datas in *DPC_ObjectDetection_ExecuteResult_t*.  
>![image](https://user-images.githubusercontent.com/85469000/190367988-c93e6130-194b-45e6-ac18-835b541ec96c.png)


2.  
In *DPC_ObjDetDSP_preStartConfig()*, declear the 4 data for later memory allocation.  
![image](https://user-images.githubusercontent.com/85469000/190368344-94090e7e-8a69-463a-87eb-8ffb7093028b.png)

Memroy allocation, both record will be allocated with space for RX ants in the first chirp:  
![image](https://user-images.githubusercontent.com/85469000/190370704-c4afea72-9dd9-4e46-9a57-71bae7f5ce77.png)


In objectdetection.h, define a new error:  
![image](https://user-images.githubusercontent.com/85469000/189861961-7684d642-39b7-4e73-9b6f-5ab7175b31e9.png)  

If the memory allocation fail, report an error:  
![image](https://user-images.githubusercontent.com/85469000/190368967-f7ea23ba-5902-4eb1-a8b6-846afa16a55c.png)


3.
Send the 4 data to *DPC_ObjDetDSP_rangeConfig*:  
![image](https://user-images.githubusercontent.com/85469000/190369165-8dcbd38d-138a-4b80-8d33-04cbe31bfff8.png)

receive it:  
![image](https://user-images.githubusercontent.com/85469000/190369378-5e8bdd13-6697-49f2-acf7-d3811ff7ec3a.png)

Load the address into hw resource:  
![image](https://user-images.githubusercontent.com/85469000/190369734-c3171662-0ec6-40b5-96f4-ca73a78aeeb1.png)
  
In *DPU_RangeProcDSP_config* -> *rangeProcDSP_ParseConfig*, load the address of 4 data from *hwRes* = *pHwRes* into *rangeProcObj* = *dpuHandle*.
![image](https://user-images.githubusercontent.com/85469000/190370242-e2691c80-93a6-4393-8a0a-f4f7f85283c6.png)

4.
In *DPU_RangeProcDSP_process()*, record the data into *ADCBufferDataRecord* for the first chirp: 
![image](https://user-images.githubusercontent.com/85469000/190371114-0ae70ab0-834b-44bb-83a9-127e386b07f6.png)

Record the FFT result:  
![image](https://user-images.githubusercontent.com/85469000/190371305-ad07eb0b-0f2f-4258-8196-be7aab3391a2.png)

5.
Load the 4 data into result buffer after range proc DPU:  
![image](https://user-images.githubusercontent.com/85469000/190371732-252b9ad5-41bf-4297-af78-dd7d4511b0ab.png)

