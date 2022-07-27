### A cfg update
Cli thread receive the updated cfg, call the *cmdHandlerFxn()* to execute the update. Take "clutterRemoval" as exaple, *CLI_task()* call *MmwDemo_CLIClutterRemoval()* to execute the update, which call *MmwDemo_CfgUpdate()* to change the cfg in *gMmwMCB.subFrameCfg[indx]* and call *MmwDemo_setSubFramePendingState()* to notify that an update is pending, by writing 1 to *subFrameCfg->objDetDynCfg.isXXXPending*.  
  
When *MmwDemo_DPC_ObjectDetection_dpmTask()* finish one *DPM_execute()*, it execute the *MmwDemo_processPendingDynamicCfgCommands()* to execute the update, which go through *subFrameCfg->objDetDynCfg.isXXXPending* and execute the change if there is an update pending.
