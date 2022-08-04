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
 
 Since the FFT function will modify the input, which is needed for zoom FFT. Thus, the input signal from ADC buffer is copied to a scratch pad:
 ![image](https://user-images.githubusercontent.com/85469000/182771078-139ed70e-1cf4-4b58-89b5-775d846a2823.png)

1D FFT:
![image](https://user-images.githubusercontent.com/85469000/182771147-3e614150-0253-4c30-a0e5-156059189c4c.png)

Then, find 3 highest peaks in the 1D FFT result, stored in *magArray[positionFlag]* and *peakListing[positionFlag]*.

Now start the zoom FFT computation. First, we need to explain some terminology. The coarse FFT refer to the FFT with (2pi)/(N) as unit frequency, the fine FFT refer to the FFT with (2pi)/(N\*N) as unit frequency.  

Here, the objective is to compute the zoom FFT with equation equal to:
$$\tilde{X}[k_{f}]=\sum_{n=0}^{N-1}\tilde{x}[n]e^{-j(\frac{2\pi}{N})(k_{0}+\frac{k_{f}}{N}) n}$$  
as explained ealier. Denote the *fft1DSize* = *nSamplesPerChirp* as *N*, using the variable inside the program, the equation can be written as:
$$\tilde{X}[j]=\sum_{k=0}^{N-1}\tilde{x}[k]e^{-j[(\frac{2\pi}{N})i\times k+\frac{2\pi}{N\times N}j\times k ]} $$
For a specific coarse FFT bin *i*, it compute the FFT with frequency (2pi/N\*i + 1\*2pi/N^2), (2pi/N\*i + 2\*2pi/N^2), (2pi/N\*i + 3\*2pi/N^2), (2pi/N\*i + 4\*2pi/N^2)...(2pi/N\*i + j\*2pi/N^2), in total N points. For each frequency, it sum up the *inputPtr[k] x ex(j\*freq.\*k)*.  
  
![image](https://user-images.githubusercontent.com/85469000/182783857-c396c852-bcc6-4032-8691-99bddb5fa609.png)
In the euqation, the i\*k is *tempCoarseSearchIdx*, the j\*k is *tempFineSearchIdx*. If the j\*k exceed the fft1Dsize, the exceeded part will be added to the *tempCoarseSearchIdx*. The *tempIndFine1* is *tempFineSearchIdx* % fft1Dsize, which is the j\*k part. The *tempIndFine2* is *tempFineSearchIdx* / fft1dsize, which is the exceeded part, which will be added to i\*k. *tempCoarseSearchIdx* is the i\*k, it is initialized as 0 and be added i for each loop. *tempIndCoarse* is the i\*k plus the exceeded part from the fine FFT.  

The *wncPtr* and *wnfPtr* are *exp(-2 * pi / N)[k]* and *exp(-2 * pi / N^2)[k]* initialized earlier:
![image](https://user-images.githubusercontent.com/85469000/182783648-78d24f49-d31e-4a39-9680-852d119985ef.png)

Here perform the actuall FFT, the f2input is first get the input from *&inputPtr[k]*, which is then multiplied with the corresponding *exp(j \* -2pi/N \*tempIndCoarse)* and *exp(j \* -2pi/N^2 \* tempIndFine1)*. The result is accumulated in to *sigAcc*.
![image](https://user-images.githubusercontent.com/85469000/182783920-c5840bb0-6e9d-4848-ba8a-c399d49e65e0.png)

After each iteration, the accumulated result's aboslut value is calculated and the max result is recorded. The *itemp* record the fine FFT index starting from the *zoomStartInd*.
![image](https://user-images.githubusercontent.com/85469000/182784555-6be9dcae-294a-4dd1-87da-fc31b8216daf.png)

After the peak is found, we need to calculate the range according to the frequency of the peak. The *interpIndx* is replated to some finetune between the previous, max and next pin, this should not matter very much. The *fdelta* is *maxBeatFreq* / *fft1DSize ^ 2*. The *maxBeatFreq* should be the sampling frequency of the ADC, so the *fdelta* is the frequency between 2 fine FFT points. *freqFineEst* is then *fdelta* x (*fft1IDsize* x *zoomStartInd* + *fineRangeInd*), which should be the frequency of the peak found by the fine FFT. The time between the start of the chirp is the freq. of the peak divided by the slop of the chirp, which is *freqFineEst* x *chirpRampTime* / *chirpBandwidth*. The distance of the object is then the time of fly times the velocity of the light divided by 2.
![image](https://user-images.githubusercontent.com/85469000/182800771-f308b669-e3b2-4a35-9815-0e9b23ac7a01.png)




