# mmwave-radar-from-first-taste-to-give-up
I am a student who try to use mmwave radar from Texas Instrument (TI) to do some projects. I will try to formulate some of my understanding here and they may help others.

I mainly use IWR6843AOP and it's evaluation module. I also use MMWAVEICBOOST for debugging.

## Advices for beginners
When starting to learn the MMWave radar, the biggest problem a beginner faces is finding and understanding documents. It was very hard for me to find the document I wanted and there were to many nouns that I didn't understand. Luckily, there are a lot of friendly engineers and advisors in E2E form, they know the radar system very well and often can answer my questions timely. If you are facing some problems as a beginner, please visit the [E2E form](https://e2e.ti.com/) and find whether others have met the same problem before. Don't hesitate to ask for help if your problem is new.

## Reference and sources
Most of the knowledge covered here are read from documents produced by TI. They have done an excellent job creating these documents, credits to them.  
  
Here I wish to introduce where you can find these documents. The documents are introduced in a recommended reading order. If you are a beginner, you may want to read through the documents following this order, or you can read through the list below and find the document you are insterested in.  
  
My recommendation is that at least read the introduction part of all documents. In this way, you can know which documents contain what information. This helps a lot when you are facing problems.  

Most of the documents can be downloaded at:  
official page for the radar [IWR6843AOP](https://www.ti.com/product/IWR6843AOP)  (The IC itself)  
page for it's evaluation module [IWR6843AOPEVM](https://www.ti.com/tool/IWR6843AOPEVM)  (The IC plus a circuit board to run and develop the radar)
  
Other documents can be found in mmwave industrial toolbox and mmwave sdk. You need to download and install them on your computer, and access the document files.  
The documents for the software development kit (SDK):   
>ti\mmwave_sdk_<ver>\docs\mmwave_sdk_module_documentation.html  
  
The documents for using Code Coposer Studio (The code editor and compilor for the radar); using MMWAVEICBOOST for debugging etc:  
>ti\mmwave_industrial_toolbox_<ver>\docs  
  
The documents for Out-Of-Box (OOB) demo:  
>ti\mmwave_industrial_toolbox_<ver>\labs\Out_Of_Box_Demo\docs  

## Recommended reading order of the documents
  
### Basic usage of the radar
1.	[mmwave sensors userguide](https://www.ti.com/lit/pdf/swru546)  
This document briefly introduces a few mmWave radar devices. Start from page 47, the document introduces the appearance of IWR6843AOPEVM. It also covers how to connect the module to a computer, how to install it on a MMWAVEICBOOST for debugging of program in the radar. This document is suitable for first receiving the radar and learn how to setup the radar for testing.  
  
2.	[mmwave sdk user guide](https://dr-download.ti.com/software-development/software-development-kit-sdk/MD-PIrUeCYr3X/03.05.00.04/mmwave_sdk_user_guide.pdf)  
This document introduces some basic operations on the radar. Firstly, the document introduces how to run the out of box demo with mmwave demo visualizer, this can give user an idea about what the radar is capable of. Next, the document introduces how to program the radar and the structure of radar program. The document is suitable for anyone that aim to modify the internal program of the radar.  
  
###	Working principle of radar
1.	[The fundamentals of millimeter wave radar sensors](https://www.ti.com/lit/pdf/spyy005)  
This document introduces the basic principle of millimeter wave radar, including how radar measure the range, velocity and the angle of objects. This is also introduced in [video from training.ti.com](https://training.ti.com/intro-mmwave-sensing-fmcw-radars-module-1-range-estimation). This document and videos are suitable for understanding basic principle of millimeter wave radar and is a must for almost all further operations on radar.  
  
2.	[MIMO Radar](https://www.ti.com/lit/pdf/swra554)  
This document further introduces how the mmwave radar with multiple receiving antennas estimate the angle of the detected object. This document is a complement for the The fundamentals of millimeter wave radar sensors, it helps to from a comprehensive understanding of the mechanism of mmWave radar.
  
###	Documents for radar data processing architectures  
1.	[mmwave sdk user guide](https://dr-download.ti.com/software-development/software-development-kit-sdk/MD-PIrUeCYr3X/03.05.00.04/mmwave_sdk_user_guide.pdf) , session 5, start from page 42  
This document introduces the architecture of the software in the mmWave radar. This includes the functionality of different layers and components in the mmWave radar software, such as data path manager, data processing chain and data processing unit. This document may not be easy to understand, but it gives a brief idea of how the system is constructed and how data is processed in the radar.  
  
2.	[introduction to the DSP Subsystem in the IWR6843](https://www.ti.com/lit/pdf/swra621)  
This document briefly introduces the function and operations in the DSP and HWA of the radar system. It corresponds to the data processing unit (DPU) in the software. This document helps to understand how algorithms such as FFT and CFAR is preformed on the data from ADC inside HWA and DSP, and how data is transferred between different algorithms. This document is a must for whom wish to modify the signal processing algorithm inside radar.  
  
###	Documents for software of the radar  
1. mmwave_sdk_module_documentation  
Located at mmwave_sdk<version>\docs. This document give links to documents of code of radar software. It includes the brief introductions for all functions and data structures. It also contains some detailed introductions of the mechanisms and workflow of demo and different modules.   
 
2. Template codes  
The template codes are located in the test folder of different modules. These templates are much smaller and simpler than the demo code and are easier to track and follow. They demo the usage of module APIs or the working principle of the modules of interest.




