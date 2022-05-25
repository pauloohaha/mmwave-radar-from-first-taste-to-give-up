# RangeProc DPU

The first step of the data processing is do the FFT on ADC samples for range calculation. Thus, Range Processng DPU is the first DPU data flow through. This DPU take RF data from ADC buffer as input, calculate the 1D FFT, and same the results in RadarCube. In this document, I will explain the test code located at *mmwave_sdk_<ver>\packages\ti\datapath\dpu\rangeproc\test* with the help of documents at *mmwave_sdk_03_05_00_04/packages/ti/datapath/dpu/rangeproc/docs/doxygen/html/index.html*.
  
  ### Reference
  1. [Further notes on EDMA](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/blob/Datapath/Further%20notes%20on%20EDMA.md#further-notes-on-edma)  
  2. [Radar Hardware Accelerator](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/blob/Datapath/Further%20notes%20on%20Radar%20Hardware%20Accelerator.md#radar-hardware-accelerator)  
  
## Code explaination
