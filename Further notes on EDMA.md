# Further notes on EDMA
EDMA is heavily used in DPUs. It is essential to understand the basic usage of EDMA. In this document, I will combine the [Enhanced Direct Memory Access (EDMA3) Controller](https://www.ti.com/lit/ug/sprugs5b/sprugs5b.pdf) and test code located in mmwave_sdk_<ver>\packages\ti\drivers\edma\test to explain the code configuration of EDMA.
  
## Code explaination
  
Like every C code, the test code start with *main()*, here initialize the SOC and start the *Test_task()*:  
  >![图片](https://user-images.githubusercontent.com/85469000/169942669-75f543ab-0314-4042-b749-79c8dc3da9b2.png)
  >![图片](https://user-images.githubusercontent.com/85469000/169942704-d58b1938-7d5f-4dba-942d-10327c1d3e31.png)

### Test_task()
  
In *Test_task()*, the code get the total number of EDMA instances on the device through *EDMA_getNumInstances()*. Lunch the *Test_instance()* on each instance.  
  >![图片](https://user-images.githubusercontent.com/85469000/169942883-02f0afef-3249-43c2-8aea-4b82eb3ecca9.png)
  
On IMR6843AOP, the *numInstances* is 2. The IWR6843AOP has 2 channel controllers (CCs), thus a *EDMAinstance* refer to a channel controller.  
  >![图片](https://user-images.githubusercontent.com/85469000/169943496-c7c730d2-4de5-4348-8a99-d471cea411a5.png)


### Test_instance()
  
In *Test_instance()*, the code first initialize and open the EDMA instance thorugh *EDMA_init()* and *EDMA_open()*.  
  >![图片](https://user-images.githubusercontent.com/85469000/169943028-41c3e23f-343c-4218-beca-d7654e9fca84.png)
