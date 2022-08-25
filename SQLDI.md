# SQL Data Insights

Beta 2.1.1
Db2 V12 Technology preview.

## USSFILES.ZFS

Create a Standard ZFS for holding pax files and matto files
To be permenantly mounted to hold USS files


```
//IBMUSERJ JOB  (NPA),'CREATE STD ZFS',CLASS=A,MSGCLASS=H,
//             NOTIFY=&SYSUID,MSGLEVEL=(1,1),REGION=0M    
//* JOB TO CREATE A STD ZFS                               
//DEFINE    EXEC   PGM=IDCAMS                             
//SYSPRINT  DD     SYSOUT=H                               
//SYSUDUMP  DD     SYSOUT=H                               
//AMSDUMP   DD     SYSOUT=H                               
//SYSIN     DD     *                                      
     DELETE 'USSFILES.ZFS'                                
     DEFINE CLUSTER (NAME(USSFILES.ZFS) -                 
            LINEAR -                                      
            TRACKS (1000 500) VOLUME(USER0A) -            
            DATACLASS(DCEXT) -                            
            SHAREOPTIONS(3))                              
/*                                                        
//FORMAT    EXEC  PGM=IOEAGFMT,REGION=0M,                 
// PARM=('-aggregate USSFILES.ZFS -compat')               
//SYSPRINT  DD     SYSOUT=H                               
//STDOUT    DD     SYSOUT=H                               
//STDERR    DD     SYSOUT=H                               
//SYSUDUMP  DD     SYSOUT=H                               
//CEEDUMP   DD     SYSOUT=H                               
//*                                                       
```

Permenantly Mount in **ADCD.Z25A.PARMLIB(BPXPRMDB)**

```
/* ----------------------------------------------------------------- */
/*                                                                   */
/* USSFILES ZFS                                                      */
/*                                                                   */
/* ----------------------------------------------------------------- */
                                                                       
MOUNT    FILESYSTEM('USSFILES.ZFS')                                    
         TYPE(ZFS)                                                     
         MODE(RDWR)                                                    
         MOUNTPOINT('/u/ibmuser/ussfiles')                             
```

Temporary Mount from ssh

mount -f USSFILES.ZFS /u/ibmuser/ussfiles

## SQLDI Beta211 Code

C:\Users\neale\code
```
-a----        15/07/2022  12:50 PM           7360 AIDB0211.IMAGE.AIDBDBRM.SEQ
-a----        15/07/2022  12:50 PM         325280 AIDB0211.IMAGE.AIDBLOAD.SEQ
-a----        06/07/2022   2:10 PM        3773360 AIDBSAMP.SEQ
-a----        15/06/2022  11:17 AM       38158848 aie_4-20-22.pax
-a----        15/06/2022  11:17 AM      319689216 sql-data-insights.pax
```

FTP the Pax Files

```
ftp 192.168.1.191
IBMUSER/SYS1
bin
put aie_4-20-22.pax /u/ibmuser/ussfiles/aie_4-20-22.pax
put sql-data-insights.pax /u/ibmuser/ussfiles/sql-data-insights.pax
```

ftp the PDS Images from Matto
```
ftp 192.168.1.191
IBMUSER/SYS1
Bin 
QUOTE SITE LRECL=80 RECFM=FB CYL PRI=5 SEC=1
put AIDB0211.IMAGE.AIDBDBRM.SEQ 'AIDB0211.IMAGE.AIDBDBRM.SEQ'
put AIDB0211.IMAGE.AIDBLOAD.SEQ 'AIDB0211.IMAGE.AIDBLOAD.SEQ'
put AIDBSAMP.SEQ 'AIDBSAMP.SEQ'
```

TSO Option 6 - Receive the PDS Images

```
RECEIVE INDA('AIDB0211.IMAGE.AIDBDBRM.SEQ')
DA('AIZ.AIDB0021.INSTALL.AIDBDBRM')
```

![aidbdbrm](images/aidbdbrm.jpg)

```
RECEIVE INDA('AIDB0211.IMAGE.AIDBLOAD.SEQ')
DA('AIZ.AIDB0021.INSTALL.AIDBLOAD')
```

![aidbload](images/aidbload.jpg)

```
RECEIVE INDA('AIDBSAMP.SEQ')
DA('AIZ.AIDB0021.INSTALL.AIDBSAMP')
```

![aidbsamp](images/aidbsamp.jpg)


## Users and Groups

Create a user AIDBADM. In a nutshell, you need to setup the following:
1. A RACF userid with an omvs segment to be the SQLDI instance owner.
2. which has generous CPU and Memory limits to reflect the fact that model training might take some time.
3. which is a member of a RACF group named SQLDIGRP.
4. and has USS environment settings that include PATH and LIBPATH values to link to all the Z AI libraries and the Deep Learning Compiler.

Copy **ADCD.JCL.LIB(ADDIDS)** to **IBMUSER.CNTL(ADDID)**

```
//IBMUSERJ JOB 'U','ADD NEW USERID',MSGCLASS=H,MSGLEVEL=(1,1),     
//         REGION=4M,TIME=1440,NOTIFY=&SYSUID,CLASS=A              
//**************************************************************** 
//*           THIS JOB needs to have caps off for OMVS seg       * 
//**************************************************************** 
//NEWID    EXEC PGM=IKJEFT01,DYNAMNBR=75,TIME=100,REGION=6M        
//SYSPRINT DD SYSOUT=*                                             
//SYSTSPRT DD SYSOUT=*                                             
//SYSTERM  DD DUMMY                                                
//SYSUADS  DD DSN=SYS1.UADS,DISP=SHR                               
//SYSLBC   DD DSN=SYS1.BRODCAST,DISP=SHR                           
//SYSTSIN  DD *                                                    
  AU AIDBADM NAME('AIDBADM') PASSWORD(AIDBADM)             -       
   OWNER(SYS1) DFLTGRP(SYS1) UACC(READ) OPERATIONS SPECIAL   -     
   TSO(ACCTNUM(ACCT#) PROC(DBSPROCC) JOBCLASS(A) MSGCLASS(X) -     
      HOLDCLASS(X) SYSOUTCLASS(X) SIZE(4048) MAXSIZE(0))     -     
    OMVS(HOME(/u/aidbadm) PROGRAM(/bin/sh) UID(1111))                 
  PERMIT ACCT#     CLASS(ACCTNUM) ID(AIDBADM)                      
  PERMIT ISPFPROC  CLASS(TSOPROC) ID(AIDBADM)                      
  PERMIT DBSPROC   CLASS(TSOPROC) ID(AIDBADM)                      
  PERMIT JCL       CLASS(TSOAUTH) ID(AIDBADM)                      
  PERMIT OPER      CLASS(TSOAUTH) ID(AIDBADM)                      
  PERMIT ACCT      CLASS(TSOAUTH) ID(AIDBADM)                      
  PERMIT MOUNT     CLASS(TSOAUTH) ID(AIDBADM)                      
  AD 'AIDBADM.*'  OWNER(AIDBADM) UACC(READ) GENERIC                
  SETROPTS REFRESH RACLIST(TSOPROC)                                
LOGOFF                                                             
```


Edit OMVS properties of AIDBADM

![aidbadm_omvs](images/aidbadm_omvs.jpg)

Display properties of AIDBADM

![aidbadm_display](images/aidbadm_display.jpg)


Edit **/u/aidbadm/.profile**

```
#export JAVA_HOME=/usr/lpp/java/J8.0_64
export _BPXK_AUTOCVT=ON
#export PATH=$PATH:/apps/zospt/bin:/usr/lpp/java/J8.0_64/bin
#export TR_Options="noResumableTrapHandler"
alias db2="java com.ibm.db2.clp.db2"
CLPHOME=/usr/lpp/db2c10/base
PATH=$PATH:/apps/zospt/bin:/usr/lpp/java/J8.0_64/bin
PATH=/usr/lpp/db2c10/jdbc/bin:$PATH
export PATH
export LIBPATH=/usr/lpp/db2c10/jdbc/lib:$LIBPATH
CLASSPATH=$CLASSPATH:/usr/lpp/db2c10/base/classes:$CLPHOME/lib/clp.jar
CLASSPATH=$CLASSPATH:/usr/lpp/db2c10/jdbc/classes/db2jcc4.jar
CLASSPATH=$CLASSPATH:/usr/lpp/db2c10/jdbc/classes/db2jcc_javax.jar
CLASSPATH=$CLASSPATH:/usr/lpp/db2c10/jdbc/classes/sqlj4.zip
CLASSPATH=$CLASSPATH:/usr/lpp/db2c10/jdbc/classes/db2jcc_license_cisuz.jar
export CLASSPATH
# ZOAU Requirements
export ZOAU_HOME=/usr/lpp/IBM/zoautil
export PATH=${ZOAU_HOME}/bin:$PATH
#ZOAU MAN
export MANPATH=${ZOAU_HOME}/docs/%L:$MANPATH
export CLASSPATH=${ZOAU_HOME}/lib/*:${CLASSPATH}
export LIBPATH=${ZOAU_HOME}/lib:${LIBPATH}
# IBM Python - Ansible Supported
export PATH=/usr/lpp/IBM/cyp/v3r9/pyz/bin:$PATH
export PYTHONPATH=/usr/lpp/IBM/cyp/v3r9/pyz
export PYTHONPATH=${PYTHONPATH}:${ZOAU_HOME}/lib

echo "Setup for SQL Data Insights Beta 2.11"

export JAVA_HOME=/usr/lpp/java/J8.0_64
export SQLDI_INSTALL_DIR=/u/ibmuser/smpework
export ZADE_INSTALL_DIR=/u/ibmuser/smpework/aie/zade
export ZAIE_INSTALL_DIR=/u/ibmuser/smpework/aie
export BLAS_INSTALL_DIR=/u/ibmuser/smpework/aie/blas
export SPARK_HOME=$SQLDI_INSTALL_DIR/spark24x

#PATH
PATH=/bin:$PATH
PATH=$SQLDI_INSTALL_DIR/sql-data-insights/bin:$PATH
PATH=$SQLDI_INSTALL_DIR/tools/bin:$PATH
PATH=$ZADE_INSTALL_DIR/bin:$PATH
PATH=$PATH:$JAVA_HOME/bin
export PATH=$PATH

#LIBPATH
LIBPATH=/lib:/usr/lib
LIBPATH=$LIBPATH:$JAVA_HOME/bin/classic
LIBPATH=$LIBPATH:$JAVA_HOME/bin/j9vm
LIBPATH=$LIBPATH:$JAVA_HOME/lib/s390x
LIBPATH=$LIBPATH:$SPARK_HOME/lib
LIBPATH=$BLAS_INSTALL_DIR/lib:$LIBPATH
LIBPATH=$ZAIE_INSTALL_DIR/zade/lib:$LIBPATH
LIBPATH=$ZAIE_INSTALL_DIR/zdnn/lib:$LIBPATH
LIBPATH=$ZAIE_INSTALL_DIR/zaio/lib:$LIBPATH
export LIBPATH=$LIBPATH

# Other system environment variables
export IBM_JAVA_OPTIONS="-Dfile.encoding=UTF-8"
export _BPXK_AUTOCVT=ON
export _BPX_SHAREAS=NO
export _ENCODE_FILE_NEW=ISO8859-1
export _ENCODE_FILE_EXISTING=UNTAGGED
export _CEE_RUNOPTS="FILETAG(AUTOCVT,AUTOTAG) POSIX(ON)"
```

Create SQLDI Group and Add Members **IBMUSER.CNTL(SQLDIGRP)**

```
//IBMUSERJ JOB  (NPA),'CREATE SQLDIGRP',CLASS=A,MSGCLASS=H,
//             NOTIFY=&SYSUID,MSGLEVEL=(1,1),REGION=0M     
//* JOB TO CREATE SQLDI GROUP                              
//S1       EXEC PGM=IKJEFT01                               
                                                           
//SYSTSPRT DD SYSOUT=*                                     
                                                           
//SYSPRINT DD SYSOUT=*                                     
                                                           
//SYSTSIN  DD *                                            
                                                           
ADDGROUP SQLDIGRP OMVS(AUTOGID) OWNER(IBMUSER)             
                                                           
CONNECT (AIDBADM) GROUP(SQLDIGRP) OWNER(IBMUSER)           
                                                           
CONNECT (IBMUSER) GROUP(SQLDIGRP) OWNER(IBMUSER)           
                                                           
SETROPTS RACLIST(FACILITY) REFRESH                         
                                                           
/*                                                         
```



## Mount a Large ZFS

Check mounted filesystems with command d omvs,f**

temporarily use SMPEWORK.ZFS mounted at /u/ibmuser/smpework 


create profile_original
create profile_sqldi 

cp profile_sqldi .profile

```
echo "Setup for SQL Data Insights Beta 2.11"        
                                                   
export JAVA_HOME=/usr/lpp/java/J8.0_64             
export SQLDI_INSTALL_DIR=/u/ibmuser/smpework               
export ZADE_INSTALL_DIR=/u/ibmuser/smpework/aie/zade       
export ZAIE_INSTALL_DIR=/u/ibmuser/smpework/aie            
export BLAS_INSTALL_DIR=/u/ibmuser/smpework/aie/blas       
export SPARK_HOME=$SQLDI_INSTALL_DIR/spark24x      
                                                   
#PATH                                              
PATH=/bin:$PATH                                    
PATH=$SQLDI_INSTALL_DIR/sql-data-insights/bin:$PATH
PATH=$SQLDI_INSTALL_DIR/tools/bin:$PATH            
PATH=$ZADE_INSTALL_DIR/bin:$PATH                   
PATH=$PATH:$JAVA_HOME/bin                          
export PATH=$PATH                                  
                                                   
#LIBPATH                                           
LIBPATH=/lib:/usr/lib                              
LIBPATH=$LIBPATH:$JAVA_HOME/bin/classic            
LIBPATH=$LIBPATH:$JAVA_HOME/bin/j9vm               
LIBPATH=$LIBPATH:$JAVA_HOME/lib/s390x              
LIBPATH=$LIBPATH:$SPARK_HOME/lib                   
LIBPATH=$BLAS_INSTALL_DIR/lib:$LIBPATH             
LIBPATH=$ZAIE_INSTALL_DIR/zade/lib:$LIBPATH        
LIBPATH=$ZAIE_INSTALL_DIR/zdnn/lib:$LIBPATH        
LIBPATH=$ZAIE_INSTALL_DIR/zaio/lib:$LIBPATH        
export LIBPATH=$LIBPATH                                                   
                                                        
# Other system environment variables                       
export IBM_JAVA_OPTIONS="-Dfile.encoding=UTF-8"            
export _BPXK_AUTOCVT=ON                                    
export _BPX_SHAREAS=NO                                     
export _ENCODE_FILE_NEW=ISO8859-1                          
export _ENCODE_FILE_EXISTING=UNTAGGED                      
export _CEE_RUNOPTS="FILETAG(AUTOCVT,AUTOTAG) POSIX(ON)"   
                                                           
# Additional environment variables and aliases             
export TERM=xterm                                          
alias vi1='vi -W filecodeset=utf-8'                        
alias vi2='vi -W filecodeset=iso8859-1'                    
alias ll='ls -ltcpa'                                       
export PS1=' ${PWD} >'                                                                             
```







Step 3: RACF Keyring
AIZ.AIDB0211.HOLFILES(RACFKR00)
AIZ.AIDB0211.HOLFILES(RACFVIEW)

Step 4: Create the Db2 artefacts
AIZ.AIDB0211.INSTALL.AIDBSAMP(DSNTIJAI)
AIZ.AIDB0211.INSTALL.AIDBSAMP(AIDBUDFC)
AIZ.AIDB0211.INSTALL.AIDBSAMP(AIDBBIND)

Step 5: Install SQLDI
mkdir /u/aidb0020/aie
/u/aidb0020/aie >pax -r -ppx -f /u/aidb0020/HOL/aie_4-20-22.pax
/u/aidb0020 >pax -r -ppx -f /u/aidb0020/HOL/sql-data-insights.pax
sqldi.sh create
Enter the keyring name: WMLZRING
Enter the keyring owner: AIDBADM
Enter certificate label: WMLZCert_WMLZID


Test:
sqldi.sh start_spark
sqldi.sh start
Open the SQLDI Web UI https://172.20.64.16:15001
Add Connection
Enable AI for CHURN
Query


