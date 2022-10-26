# ZOWE Update

ZWE100.CSI

Run Report for ZWE100.GLOBAL.CSI

```
//IBMUSERJ JOB  (NPA),'INIT 3380 DASD',CLASS=A,MSGCLASS=H,   
//             NOTIFY=&SYSUID,MSGLEVEL=(1,1),REGION=0M       
//* GENERATE INSTALLED SOFTWARE REPORT                       
//STEP1    EXEC PGM=GIMXSID,PARM='WAIT=10MIN,L=ENU'          
//SYSPRINT DD SYSOUT=*                                       
//SMPOUT   DD SYSOUT=*                                       
//SMPXTOUT DD PATH='/u/ibmuser/reports/zwe.report',          
//            PATHOPTS=(OWRONLY,OCREAT,OTRUNC),              
//            FILEDATA=BINARY,PATHMODE=(SIRWXU,SIRWXG,SIRWXO)
//SYSIN    DD DATA,DLM=$$                                    
CSI=ZWE100.GLOBAL.CSI                                        
TARGET=ZWE100T                                               
$$                                                           
```

Order Everything newer with report from ShopZ

