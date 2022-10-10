# Classic CDC for IMS Customisation.

Goals : 
1. Prove the Serverpac / PSI Method works
2. Well placed to build the image for ZVA
3. Well Placed to build the Db2 Roadshow content

## Post Install

Looks Familiar

```
CB.ST251564.CAC.SCACBASE                                       USER0E
CB.ST251564.CAC.SCACCONF                                       USER0E
CB.ST251564.CAC.SCACLOAD                                       USER0E
CB.ST251564.CAC.SCACMAC                                        USER0E
CB.ST251564.CAC.SCACMSGS                                       USER0E
CB.ST251564.CAC.SCACSAMP                                       USER0E
CB.ST251564.CAC.SCACSIDE                                       USER0E
CB.ST251564.CAC.SCACSKEL                                       USER0E
```


CB.ST251564.CAC.SCACSAMP(CECCUSC1)

```
//GENSAMP PROC CAC='CB.ST251564.CAC',
//             DISKU='SYSALLDA',     
//             DISKVOL='USER0A',     
//             ISPHLQ='ISP',         
//             ISPLANG='ENU'         
//*                                  
...
//CRTSAMP   EXEC PGM=IKJEFT1A,REGION=0M                     
//STEPLIB   DD DISP=SHR,DSN=&ISPHLQ..SISPLOAD               
//          DD DISP=SHR,DSN=&ISPHLQ..SISPLPA                
//          DD DISP=SHR,DSN=&CAC..SCACLOAD                  
//SYSOUT    DD SYSOUT=*                                     
//SYSPRINT  DD SYSOUT=*                                     
//SYSTSPRT  DD SYSOUT=*                                     
//ISPFTTRC  DD SYSOUT=*,DCB=(LRECL=256,RECFM=VB)            
//ISPPROF   DD DSN=&&PROF,                                  
//             DISP=(NEW,DELETE),                           
//             UNIT=&DISKU,                                 
//             SPACE=(CYL,(1,,10)),                         
//             DCB=(LRECL=80,BLKSIZE=6160,RECFM=FB,DSORG=PO)
...
//CRTSAMP.SYSTSIN  DD *                                          
 PROFILE NOPREFIX  MSGID                                         
                                                                 
 ISPSTART CMD(%CACCUSX1                                         +
      CACDUNIT=SYSALLDA                                         +
  /*  CACDVOLM=<CACDVOLM>         UNCOMMENT IF NEEDED  */       +
  /*  CACMGTCL=<CACMGTCL>         UNCOMMENT IF NEEDED  */       +
  /*  CACSTGCL=<CACSTGCL>         UNCOMMENT IF NEEDED  */       +
      CACINHLQ=CB.ST251564.CAC                                  +
      CACUSHLQ=CB.ST251564.CAC.I1                               +
      ISPFHLQ=ISP                                               +
      ISPFLANG=ENU                                              +
      SERVERROLE=(CDC_IMS_SRC)                                  +
 )                                                               
 ```
 
 

