## RF components of Mmwave radar
The RF part of the radar cannot be programmed directly by user. It can only be controlled by setting some parameters and call some APIs. Below I will describe the principle of some parameters in RF settings.  
  
Source:  
[Introduction to mmwave Sensing: FMCW Radars](https://training.ti.com/sites/default/files/docs/mmwaveSensing-FMCW-offlineviewing_0.pdf)  
[Programming Chirp Parameters in TI Radar Devices](https://www.ti.com/lit/pdf/swra553)  

## Maximum range  
The EM wave need to travel from radar to the object and back to radar. If the time needed for the travel is longer than the chirp, the object cannot be detected. Let the bandwidth of the IF signal be IF<sub>MAX</sub>
