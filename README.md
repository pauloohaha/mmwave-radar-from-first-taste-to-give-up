# mmwave-radar-from-first-taste-to-give-up
I am a student who try to use mmwave radar from Texas Instrument (TI) to do some projects. I will try to formulate some of my understanding here and they may help others.

If you are a beginner, I will recommend you go through the [learning guide](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/blob/2366bb83f49f4aefdf32ee61b63d935c68d2a57e/Learining%20Guide%20and%20reference.md). I introduce some basic idea and put some useful documents there.

I mainly use IWR6843AOP and it's evaluation module. I also use MMWAVEICBOOST for debugging.

## Advices for beginners
When starting to learn the MMWave radar, the biggest problem a beginner faces is finding and understanding documents. It was very hard for me to find the document I wanted and there were to many nouns that I didn't understand. Luckily, there are a lot of friendly engineers and advisors in E2E forum, they know the radar system very well and often can answer my questions timely. If you are facing some problems as a beginner, please visit the [E2E forum](https://e2e.ti.com/) and find whether others have met the same problem before. Don't hesitate to ask for help if your problem is new.
  
## About
In this repository, I will not cover everthing about the radar. I am not capable of that and I think most documents are easy to understand. But, I will introduce to you, from my own experience, where to start, what to read and in which order. In the [Learning Guide and Reference](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/blob/main/Learining%20Guide%20and%20reference.md#learining-guide-and-reference) file, I will briefly introduce the radar system. Then guide you through some improtant documents that you may wish to read first.  

In the [Datapath](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/tree/Datapath#datapath) branch, I will summarize the information from *mmwave_sdk_module_documentation* and provide my own understandings about the processing of the data in the radar system. I explain the test codes provided by TI for HWA, EDMA and RangeProc DPU will relavent documents.  

In the [RF](https://github.com/pauloohaha/mmwave-radar-from-first-taste-to-give-up/tree/RF#rf-components-of-mmwave-radar) branch, I will explain the setting of RF components.

## Reference and sources
Most of the knowledge covered here are read from documents produced by TI. They have done an excellent job creating these documents, credits to them.  

Most of the documents can be downloaded at:  
page for the radar [IWR6843AOP](https://www.ti.com/product/IWR6843AOP)  (The IC itself)  Here contains documents about the operating principle and working mechanisms.  
page for it's evaluation module [IWR6843AOPEVM](https://www.ti.com/tool/IWR6843AOPEVM)  (The IC plus a circuit board to run and develop the radar) Here only has documents about the whole modules, such as circuit board design and so on.
  
Other documents can be found in mmwave industrial toolbox and mmwave sdk. You need to download and install them on your computer, and access the document files. Or they can also be accessed through [TI resource explorer](https://dev.ti.com/).
The documents for the software development kit (SDK):   
>ti\mmwave_sdk_<ver>\docs\mmwave_sdk_module_documentation.html  
  
The documents for using Code Coposer Studio (The code editor and compilor for the radar); using MMWAVEICBOOST for debugging etc:  
>ti\mmwave_industrial_toolbox_<ver>\docs  
  
The documents for Out-Of-Box (OOB) demo:  
>ti\mmwave_industrial_toolbox_<ver>\labs\Out_Of_Box_Demo\docs  



