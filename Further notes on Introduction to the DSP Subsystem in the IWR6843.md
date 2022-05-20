I put the rest of my notes on *Introduction to the DSP Subsystem in the IWR6843* here, since I think they are less relevant to the usage of HWA. However, they are still helpful for understanding the Data Processing Chain.  
  
## Radar Signal processing Chain
The data processing chain mainly consists of 3 FFTs. Range-FFT, doppler-FFT and angle-FFT. The range-FFT do FFT on ADC samples of one chirp of one RX antenna with respect to time, thus get the frequency spectrum of the received signal of one chirp to resolve time. Doppler-FFT do FFT on one RX antenna's multiple chirps' rang-FFT results with respect to time, to detect the phase change at different range with respect to time, thus resolve the velocity of object relative to radar. Angle-FFT do FFT on the different RX antennas' doppler FFT result, to resolve the angle of the objects. They will be described more clearly in the next section.  
  
The fundamental transmission unit of an FMCW radar is a frame, which consists of multiple chirps. Range-FFT can be performed once the chirp is finished. Thus, they are performed during *inrea-frame* time. Doppler-FFT can only be performed after all range-FFT of all chirps are done. Angle-FFT can only be performed after doppler-FFT, so both of them are done after each frame is done.  
  
There are 3 similar signal processing chains. There difference between first 2 are 16-bit or 32-bit doppler-FFT result. 32-bit allow higher percision but at a cost of time. The 3rd uses floating point numbers after doppelr-FFT. I will only introduce 16-bit bellow, you can refer to [Introduction to the DSP Subsystem in the IWR6843](https://www.ti.com/lit/an/swra621/swra621.pdf) for other types.  
  
## 16-bit Processing Chain
![图片](https://user-images.githubusercontent.com/85469000/169474212-51a1f757-1078-4fdc-913b-7d97c4f84bbb.png)
The diagram above shows a typical data processing chain in FMCW radar for each frame. The ADC samples of all 4 RX antennas are placed in the ADC buffer, then processed by FFTs and other algorithms. The different color of the blocks indicate on HWA or DSP or both should these algorithms be performed.  
  
The first setp is perform an *interference mitigation* algorithm. The output is then sent to HWA to performs FFTs for each RX antenna to do the range-FFTs. The result is placed in L3 memory. The result can be visualized by a 3 dimensional radar cube. With range, antennas and chirps as 3 axises.  
  
After the range-FFTs for all chirps are done, doppler-FFT can be performed. For each range-bin and each RX antenna, a Doppler-FFTis performed across N chirps. Since in the 16-bit processing chain, both the results of range-FFT and doppler-FFT are 16-bits. The output of doppler-FFT can overwrite the range-FFT samples in L3. After doppler-FFT the 3 dimensional radar cube is indexed along range, doppler and antennas.
  
After doppler-FFT is done, the radar creats a *pre-detection array*. It summs the doppler-FFT outputs along the antenna dimension, which means it summs the datas at same range bin and same doppler bin but from different antennas into one data. It also turns the data into real number, thus shrinking the size of radar cube by 1/8.  
  
Then, the detection algorithm is run on the *pre-detection array*. If the algorithm is CFAR-CA supported by the HWA, it can be run on HWA. Otherwise, HWA should interrupt DSP after doppler-FFT is done and DSP do the detection. Two detection algorithms are run along doppler direction and range direction and creat a detected objects list.  
  
With the detected objects list, a angle -FFT is done to compute the angle of arrival of the object. Finally, parameters of the object is shared to the MASS via handshake memory and other algorithms are performed.
