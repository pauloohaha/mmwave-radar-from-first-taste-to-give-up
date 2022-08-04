The *Level sensing* lab in mmavw industrial toolbox demo the high accuracy range detection capability of mmwave radar. It uses both mss and dss, the data path is mainly realized on dss.  

## Math principle

I will first introduce the mathematical principle of zoom FFT here in mmwave radar. Normally, the equation for FFT is as follow:
$$\tilde{X}[k]=\sum_{n=0}^{N-1}\tilde{x}[n]e^{-j(\frac{2\pi}{N})kn}$$   

The spacing between two point in X[k] is 2pi/N. For example, we find a local maximun at k0, but we need to know that exact frequency of the peak with higher resolution. We can realize this by performing another N-point FFT between k0 and k0+1. Thus the equation for zoom-FFT becomes:
$$\tilde{X}[k_{f}]=\sum_{n=0}^{N-1}\tilde{x}[n]e^{-j(\frac{2\pi}{N})(k_{0}+\frac{k_{f}}{N}) n}$$

Where kf represent the index for zoom FFT.  

In this way, we can get another N-point FFT between k0 and k0+1, thus be able to find the frequency of the peak more percisly.
>![bf96ace2b53bd3bbabce4f9ae59010c](https://user-images.githubusercontent.com/85469000/182756171-7d535eae-84e9-42a3-aebe-c9f320e73a48.jpg)

 
## Code realization
  In *dss_main.c*, the *MmwDemo_dssDataPathProcessEvents* is responsible for performing chirp processing. For each chirp, it call *MmwDemo_processChirp* to copy data from ADC buffer and process them:  
  ![image](https://user-images.githubusercontent.com/85469000/182518930-d8e013a1-c28e-4d9d-8e0f-fc1f0ed8344d.png)
  
  In *RADARDEMO_highAccuRangeProc_run*, it calls two functions: *RADARDEMO_highAccuRangeProc_accumulateInput* or *RADARDEMO_highAccuRangeProc_rangeEst*. The first one is responsible for accumulating ADC buffer results of all chirp and convert them into float.  
   
### Sum up the input signals from different chirps
  For the first chirp, it loop though the input adc samples and convert them to float and store them into memory. it uses *_amem8* to store 8 bytes (2 x cplx16_t) into memory, the stored value is referenced by *input*. Next, it convert the imag and real part of 2 complex number into float. It first call *_loll* or *_hill* to extract lower or higher 32 bits from the 8 bytes stored earlier. The *_ext* is used to extract lower or higher 16 bits from the 32 bits, this is done by shifting the 32 bit left by second arguement and signed shift right by third arguement, then extend to 32 bit. Then, the two int numbers are converted to float and convered to *__float2_t* type. Then stored by *_amem8_f2* into *outputPtr*. The *if((int32_t)nSamplesPerChirp & 1)* part is responsible for the last sample when the *nSamplesPerChirp* is an odd number.
  ![image](https://user-images.githubusercontent.com/85469000/182519415-f03058d6-a22a-437e-89ae-7bb1121cbcd5.png)
  
  For the chirps after the first chirp, it convrt the input to 2 float number similarly, and add it to the respective result in the *outputPtr* by *_daddsp*.  
  ![image](https://user-images.githubusercontent.com/85469000/182521422-f53929cd-609c-4651-9252-84b5848302ad.png)
  
  For the last chirp, the windowing coefficient is multiplied. It first convert the windowing coefficient into a floating point complex number, then multiply it with the sum of chirps in the *outputPtr* by *_dmpysp*. If the *fftsize1D* is bigger than the *nSamplesPerChirp*, zero padding is needed, the output is extended to the length of fftsize1D by adding 0.

### Do range FFT and zoom FFT
  
