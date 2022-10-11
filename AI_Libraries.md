# AI Libraries 

z/OS V2.5
* OA62901 zDNN  (Deep Neural Network library) 
* OA62902 zAIO  (opt'n library) 
* OA62903 zADE (data embed library) 
* OA62728 Supervisor 
* PH45672 PH44479 PH46862  OpenBLAS

SMPE Query these APARs for MVS.GLOBAL.CSI 

Run Report for MVS.GLOBAL.CSI

Order APARs with report from ShopZ
OA62901 OA62902 OA62903 OA62728 PH45672 PH44479 PH46862

```
  <SERVER                                                                       
    host="deliverycb-bld.dhe.ibm.com"                                                         
    user="S9669i02"                                                             
    pw="b99005994b7793D"                                                          
    >                                                                           
    <PACKAGE                                                                    
      file="2022101001289/PROD/GIMPAF.XML"                                                      
      hash="1EDDBC4E725467C27A305584CCC73ED9432ACC3C"                                                          
      id="U02386687"                                                            
      >                                                                         
    </PACKAGE>                                                                  
  </SERVER> 
  ```
  
  IBMUSER.CNTL(AILIBDWN)
  
  Read Job output to get list of PTFs
  UJ08004 UJ08444 UJ08448 UI80106 UI80156 UI80826 UJ08371 UJ08381
  
  Run Apply.
  
  ```
  //IBMUSERJ JOB  (PTF),'DOWNLOAD PTF',CLASS=A,MSGCLASS=H,         
//             NOTIFY=&SYSUID,MSGLEVEL=(1,1),REGION=0M           
//* APPLY OF UI79663 TO ENABLE PSI INSTALL CCDC IMS              
//RECNTS   EXEC PGM=GIMSMP,REGION=0M,                            
//         PARM='CSI=MVS.GLOBAL.CSI'                             
//SMPOUT   DD SYSOUT=*                                           
//SMPLOG   DD SYSOUT=*                                           
//SMPNTS   DD PATH='/u/ibmuser/smpework/ailib'                   
//*SMPJHOME DD PATH='/usr/lpp/java/J8.0/',PATHDISP=KEEP          
//SYSUT1   DD UNIT=SYSDA,SPACE=(CYL,(380,760)),DSNTYPE=LARGE,    
//            STORCLAS=BIGSMS                                    
//SYSUT2   DD UNIT=SYSDA,SPACE=(3120,(380,760))                  
//SYSUT3   DD UNIT=SYSDA,SPACE=(3120,(380,760))                  
//SYSUT4   DD UNIT=SYSDA,SPACE=(3120,(380,760))                  
//*SMPHOLD  DD DISP=SHR,DSN=ADCDMST.HOLDDATA.TXT                 
//SYSPRINT DD SYSOUT=*                                           
//SMPCNTL  DD *                                                  
 SET BDY(MVST  ).                                                
 APPLY   S(                                                      
 UJ08004 UJ08444 UJ08448 UI80106 UI80156 UI80826 UJ08371 UJ08381 
 )  CHECK GROUPEXTEND BYPASS(HOLDSYS)                            
 .                                                               
/*                                                               
```
  
