# Datapath

## Introduction
In this branch, I will document mainly about the data processing part of the IWR6843AOP. I will introduce the architecture of the data processing in this document, and explain each dpu and its implementation in other documents. The documents provided by TI are quit intuitive, but I think an understanding from a different perspective will be helpful. 
All information below comes from:  
1) [Introduction to the DSP Subsystem in the IWR6843](https://www.ti.com/lit/an/swra621/swra621.pdf)  
2) [Enhanced Direct Memory Access (EDMA3) Controller](https://www.ti.com/lit/ug/sprugs5b/sprugs5b.pdf)  
3) [Radar Hardware Accelerator](https://www.ti.com/lit/ug/swru526b/swru526b.pdf)  
  
## Data Path
The Data Path refer to the architecture that process the ADC sample datas from RF part and eventually extrapolate useful information.  
The diagram bellow best illustrate the structure of Data Path. The Data Path can be split into four layers. The tallest Data Path Manager (DPM) layer is responsible for providing an simple and easy API for application. The application only need to call some simple functions such as DPM_init(), DPM_start(), DPM_execute(), etc., in a proper manner, without worring about internal data transfer and processing, and can get the results.  
  
The Data Processing Chain (DPC) is the layer that cordinate different processes of the Data. It is responsible for initializing, configuration, etc., each Data Processing Unit (DPU), and present the results back to DPM.  
  
The Data Processing Unit (DPU) is the layer responsible for configurating the Hardware Accelerator (HWA) and Digital Signal Processor (DSP) for different computations. DPU need to set the correct triggering sequences, data transfer sequences, computation settings, etc..  
  
When the HWA or the DSP are configurated correctly, they will start computations once they receive the ADC samples. The Enhanced Direct Memory Access (EDMA), will help to transfer data. Thus, there is no need for the main processor to intervene during the acutal computation.
>![图片](https://user-images.githubusercontent.com/85469000/168749740-30ee4d2c-2009-47d5-a890-1e2a991a9d72.png)
*mmwave sdk userguide P50*  
  
You should refer to *mmwave sdk user guide* and the *mmwave sdk module document* for detailed sequence and functions of each module.

## Principle of DSP system on mmwave radar  
IWR6843 has a hardware accelerator (HWA) and a digital signal processor (DSP) on board. We need to know how does these 2 components work before diving into any DPU.  
  
Bellow I summerize some key information from [introduction to the DSP Subsystem in the IWR6843](https://www.ti.com/lit/pdf/swra621). 
  
### Overview
Below is the basic structuer of DSP Sub-system (DSS) and expected role of each components. It contains a DSP and a HWA.  
  
The DSP is basically and processor specialized for large amount of computations, it is optimized for large throughput of data and has great parallelism.  
  
The HWA has much limited functions. It is mainly used to do FFT, log and CFAR computations. For example do FFTs on ADC sample to calculate range (the first FFT or 1D-FFT), doppler (the second FFT or 2D-FFT) and angle (the third FFT or 3D-FFT). After programmed properly, HWA can run indepently, without processor supervision, thus reduce the load of DSP significantly.  
![图片](https://user-images.githubusercontent.com/85469000/168815921-02bae905-6ceb-4b39-91f4-133ce56bc48c.png)

### HWA
HWA has four local memories. Data can be brought into one local memory from external memory, processed by HWA, and output to another local memory. **Noted that HWA can not read and write same local memory at the same time.**  
  
Four local memory allow a special *Ping-Pong* operation. Since HWA need to read from and write to different local memories, HWA need two local memories to input and output at the same time. Meanwhile, the two other local memory can be used to output previpously computed results to and get data from external memory for next round of computation. For example, HWA is inputing data from MEM0, doing the computation, writing results to MEM2, this input and output pair is called *Ping*. At the same time, MEM1 take data from external memory for next HWA computation, MEM3 output the result of last HWA computation, this input output pair is called *Pong*.

![图片](https://user-images.githubusercontent.com/85469000/168817727-aa274abd-9420-4955-8b9f-82aa5f0c4845.png)

The HWA has three main components:  
  
The **Core Computation Unit** is the component to perform acutual computations. It is capable of FFT, pre-FFT (windowing, zero-out, etc.), CFAR, maganitude and log-magnitude computations.  
  
The **input and output formattor** interface between the input (or output) data format and the internal HWA data format. They are also responsible for reading and writing data from and into local MEM.  
  
The **State Machine**  is responsible for controlling the HWA. It can chain and loop through a sequence of parameter sets one after another. The state machine has 2 kinds of registers, (1) parameter-set memories (responsible for controlling the computation) and (2) static register (responsible for controlling the execution of parameter-set defined operations). The parameter-set memory can specify (1) which kind of operation, (2) input and output data format, (3) number of iterations this parameter set is executed, (4) the condition which trigger the execution or stalled until. The static registers define the pattern of parameter-set is executed.

You can refer to [Radar Hardware Accelerator User's Guide](http://www.ti.com/lit/pdf/SWRU526) for more details.  
  
### Enhanced-DMA (EDMA)
  
Data need to be intensivly transfered between local memory and external memory during HWA computation, these data transfers are done by EDMA. The purpose of EDMA is not only to transfer data, EDMA can also be used to do multidimensional transfer, chaining and linking multiple transfers.  
  
EDMA controller mainly consist of channel controllers (CC) and multiple tanfer coontrollers (TC) for each CC. CC is responsible for scheduling data transfers, TC is responsible for performing actual data transfers. For IWR6843, it has 2 CCs, each with 2 TCs. Thus, 4 TCs in total, so 4 data transfers can be performed in parallel.
![图片](https://user-images.githubusercontent.com/85469000/169446499-f7370280-de40-4908-a6b3-6cf43e8a9793.png)
> [Enhanced Direct Memory Access (EDMA3) Controller](https://www.ti.com/lit/ug/sprugs5b/sprugs5b.pdf)  
  
EDMA engine is programmed by parameter RAMs (PaRAM) table. The PaRAM table contains multiple RaRAM sets. Each set define a DMA transfer, including the source and destination addresses, data pattern, linking, chaining and indexing. In radar, the EDMA is mainly used at:  
(1) From the ADC Buffer into L3.  
(2) From (or to) the L1/L2 DSP to (or from) the L3 or the HWA local memories.  
(3) From (or to) the HWA local memories to (or from) the L3 or the DSP L1/L2 memories.  


