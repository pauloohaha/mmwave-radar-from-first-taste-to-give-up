# Datapath

## Introduction
In this branch, I will document mainly about the data processing part of the IWR6843AOP. I will introduce the architecture of the data processing in this document, and explain each dpu and its implementation in other documents. The documents provided by TI are quit intuitive, but I think an understanding from a different perspective will be helpful.  
  
## Data Path
The Data Path refer to the architecture that process the ADC sample datas from RF part and eventually extrapolate useful information.  
The diagram bellow best illustrate the structure of Data Path. The Data Path can be split into four layers. The tallest Data Path Manager (DPM) layer is responsible for providing an simple and easy API for application. The application only need to call some simple functions such as DPM_init(), DPM_start(), DPM_execute(), etc., in a proper manner, without worring about internal data transfer and processing, and can get the results.  
  
The Data Processing Chain (DPC) is the layer that cordinate different processes of the Data. It is responsible for initializing, configuration, etc., each Data Processing Unit (DPU), and present the results back to DPM.  
  
The Data Processing Unit (DPU) is the layer responsible for configurating the Hardware Accelerator (HWA) and Digital Signal Processor (DSP) for different computations. DPU need to set the correct triggering sequences, data transfer sequences, computation settings, etc..  
  
When the HWA or the DSP are configurated correctly, they will start computations once they receive the ADC samples. The Enhanced Direct Memory Access (EDMA), will help to transfer data. Thus, there is no need for the main processor to intervene during the acutal computation.
![图片](https://user-images.githubusercontent.com/85469000/168749740-30ee4d2c-2009-47d5-a890-1e2a991a9d72.png)
You should refer to *mmwave sdk user guide* and the *mmwave sdk module document* for detailed sequence and functions of each module.
