## RF components of Mmwave radar
The RF part of the radar cannot be programmed directly by user. It can only be controlled by setting some parameters and call some APIs. Below I will describe the principle of some parameters in RF settings.  
  
Source:  
[Introduction to mmwave Sensing: FMCW Radars](https://training.ti.com/sites/default/files/docs/mmwaveSensing-FMCW-offlineviewing_0.pdf)  
[Programming Chirp Parameters in TI Radar Devices](https://www.ti.com/lit/pdf/swra553)  

## Maximum range  
The EM wave need to travel from radar to the object and back to radar. If the time needed for the travel is longer than the chirp, the object cannot be detected. Let the bandwidth of the IF signal be IF<sub>MAX</sub>, c be the speed of the light, S be the slop of the chirp and d<sub>MAX</sub> be the maximum distance of the object. The time period of the chirp is IF<sub>MAX</sub>/S. The time needed for the travel is 2\*d<sub>MAX</sub>/c. Thus 2\*d<sub>MAX</sub>/c < IF<sub>MAX</sub>/S, d<sub>MAX</sub>
