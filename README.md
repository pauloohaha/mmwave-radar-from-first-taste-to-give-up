## RF components of Mmwave radar
The RF part of the radar cannot be programmed directly by user. It can only be controlled by setting some parameters and call some APIs. Below I will describe the principle of some parameters in RF settings.  
  
Source:  
[Introduction to mmwave Sensing: FMCW Radars](https://training.ti.com/sites/default/files/docs/mmwaveSensing-FMCW-offlineviewing_0.pdf)  
[Programming Chirp Parameters in TI Radar Devices](https://www.ti.com/lit/pdf/swra553)  

## Maximum range  
The EM wave need to travel from radar to the object and back to radar. If the time needed for the travel is longer than the chirp, the object cannot be detected. Let the   
IF<sub>MAX</sub> be bandwidth of the IF signal  
c be the speed of the light  
S be the slop of the chirp  
d<sub>MAX</sub> be the maximum distance of the object  
The time period of the chirp is IF<sub>MAX</sub>/S. The time needed for the travel is 2\*d<sub>MAX</sub>/c. Thus 2\*d<sub>MAX</sub>/c < IF<sub>MAX</sub>/S, d<sub>MAX</sub> < c\*IF<sub>MAX</sub>/2\*S  
If the SNR is also considered, the formula is expressed in P3 of [Programming Chirp Parameters in TI Radar Devices](https://www.ti.com/lit/pdf/swra553) 
  
## Range resolution
For easier resolution during range FFT, the IF signal for two objects need to have at least one period difference.  
>![图片](https://user-images.githubusercontent.com/85469000/169676410-13e1797d-bff3-4acd-b702-dc07cb850f40.png)
>[Introduction to mmwave Sensing: FMCW Radars](https://training.ti.com/sites/default/files/docs/mmwaveSensing-FMCW-offlineviewing_0.pdf) 
  
Let the  
f1 and f2 be the IF signal frequence of two objects  
delta f = |f1 - f2|  
delta d be the distance difference between 2 objects  
T be the period of the chirp  
B be the sweep bandwidth of the chirp  
|f1 \* T - f2 \* T| >= 1 to allow at least one period difference.  
delta f > 1 / T ===> S * 2 * delta d / c > 1 / T ===> delta d > c / 2 * S * T ===> delta d > c / 2 * B
  
## Maximum velocity
Let the  
T<sub>c</sub> be the chirp seperation  
v be the velocity of the object  
lambda be the wavelength  
w be the phase difference  
w = 2 * v * T<sub>c</sub> * 2 * pi / lambda  
v = w * lambda / 4 * pi * T<sub>c</sub>  
since the phase difference cannot be larger than pi, otherwise the velocity will be unambiguous.  
v < pi * lambda / 4 * pi * T<sub>c</sub> ===> lambda / 4 * T<sub>c</sub>  
  
## Velocity resolution 
Suppose there are N chirps. Thus the phase difference accross N chirps need to be larger than 2 * pi.  
w = 2 * N * v * T<sub>c</sub> * 2 * pi / lambda > 2 * pi  
v < 2 * pi * lambda / 4 * N * pi * T<sub>c</sub> = lambda / 2 * N * pi * T<sub>c</sub>  
