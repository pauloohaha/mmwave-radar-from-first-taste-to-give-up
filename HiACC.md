The *Level sensing* lab in mmavw industrial toolbox demo the high accuracy range detection capability of mmwave radar. It uses both mss and dss, the data path is mainly realized on dss.  
  
  In *dss_main.c*, the *MmwDemo_dssDataPathProcessEvents* is responsible for performing chirp processing. For each chirp, it call *MmwDemo_processChirp* to copy data from ADC buffer and process them:
  #![image](https://user-images.githubusercontent.com/85469000/182518930-d8e013a1-c28e-4d9d-8e0f-fc1f0ed8344d.png)
  
  In *RADARDEMO_highAccuRangeProc_run*, it calls two functions: *RADARDEMO_highAccuRangeProc_accumulateInput* or *RADARDEMO_highAccuRangeProc_rangeEst*. The first one is responsible for accumulating ADC buffer results of all chirp and convert them into float.  
   
  For the first chirp, it uses *_amem8* to store 8 bytes (2 x cplx16_t) into memory, the stored value is referenced by *input*. Next, it convert the imag and real part of 2 complex number into float. It first call *_loll* or *_hill* to extract lower or higher 32 bits from the 8 bytes stored earlier. The *_ext* is used to extract lower or higher 16 bits from the 32 bits, this is done by shifting the 32 bit left by second arguement and signed shift right by third arguement, then extend to 32 bit. Then, the two int numbers are converted to float and convered to *__float2_t* type. Then stored by *_amem8_f2*.
  #![image](https://user-images.githubusercontent.com/85469000/182519415-f03058d6-a22a-437e-89ae-7bb1121cbcd5.png)

  
