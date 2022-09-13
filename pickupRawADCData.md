# Pick up raw ADC data
## Idea
Record the adc buffer data for the first chirp only.  

Similar to radar cube, define a *cmplx16ImRe_t* type *ADCBufferDataRecord* array in *DPC_ObjectDetection_ExecuteResult_t*, allocate memory to it at *DPC_ObjDetDSP_preStartConfig*, send its address to range processing DPU through *DPC_ObjDetDSP_rangeConfig*, plug it into *rangeCfg.hwRes*, which is the *pConfig* in *DPU_RangeProcDSP_config*. Then, the *pConfig* is send into *rangeProcDSP_ParseConfig*, where the address of the *ADCBufferDataRecord* can be stored into *rangeProcObj*. It will be passed to *DPU_RangeProcDSP_process* later, in which the adc data can be copied into the *ADCBufferDataRecord*.

## Modification made
1.  
In *objectdetecion.h*, define a *ADCBufferDataRecord* in *DPC_ObjectDetection_ExecuteResult_t*.  
>![image](https://user-images.githubusercontent.com/85469000/189856847-b9f92dd6-dd27-4868-a8a5-97aeb59efd80.png)  

2.  
In *DPC_ObjDetDSP_preStartConfig()*, define a *ADCBufferDataRecord* and allocate memory for it.  
>![image](https://user-images.githubusercontent.com/85469000/189859158-33eef2ac-fad1-406f-a7c4-39b224f76516.png)  

Memroy allocation:  
>![image](https://user-images.githubusercontent.com/85469000/189861859-829fe1dd-23fd-420c-a227-6ca55b1dbc0d.png)  

In objectdetection.h, define a new error:  
![image](https://user-images.githubusercontent.com/85469000/189861961-7684d642-39b7-4e73-9b6f-5ab7175b31e9.png)  

3.
Send the *ADCBufferDataRecord* to *DPC_ObjDetDSP_rangeConfig*:  
>![image](https://user-images.githubusercontent.com/85469000/189862610-fec12695-fcb4-4be2-b808-fddad4639d72.png)  

receive it:  
![image](https://user-images.githubusercontent.com/85469000/189862742-63489967-1f20-4794-8734-c45a0e970736.png)  

In *rangeprocdsp.h* -> *DPU_RangeProcDSP_HW_Resources_t*, define a new data field *ADCBufferDataRecord*:  
>![image](https://user-images.githubusercontent.com/85469000/189862817-343f41b7-1658-4d2b-a0a9-2e7b53e3db7e.png)  

Load the address into hw resource:  
![image](https://user-images.githubusercontent.com/85469000/189863190-8899b1ae-ff90-4f16-b16c-339719a6a3f5.png)

4.  
In *rangeprocdsp_internal.h* -> *rangeProcDSPObj_t*, define a new data field:  
>![image](https://user-images.githubusercontent.com/85469000/189863828-5458e673-e158-411d-a006-5222e59ff8e8.png)

In *rangeProcDSP_ParseConfig*, load the address of *ADCBufferDataRecord* from *hwRes* = *pHwRes* into *rangeProcObj* = *dpuHandle*.
