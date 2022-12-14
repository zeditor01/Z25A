# Basic Customisations.

## IPL

firstipl.sh 
awsckmap devmap.txt
awsstart --map devmap.txt
x3270 -port 3270 mstcon@localhost &
x3270 -port 3270 tso@localhost &
ipl 0a80 parm 0A82CS

Respond to Master Console Prompts with **r 0,i** on first IPL


![firstipl](images/firstipl.png)

## Logon

IBMUSER ( IBMUSER )
Change PWD to ( SYS1 )
Logn Proc DBSPROCC


## Clock

Set the Clock .... Edit ADCD.Z24C.PARMLIB because the USER.Z24C.PARMLIB concatenation does NOT work

Member **IEASYSDB** and **IASSYSAL** both point to CLOCK00

![ieasysdb](images/ieasysdb.jpg)

Set the clock to Sydney by editing **ADCD.Z25A.PARMLIB(CLOCK00)**

![clock00](images/clock00.jpg)



## PARMLIB and PROCLIB Concatenations

Parmlib concatenations for the DB IPL Parm defined in **SYS1.IPLPARM(LOADDB)**

![parmlib_concat](images/parmlib_concat.jpg)

Proclib Concatenation defined in **ADCD.Z25A.PARMLIB(MSTJCL00)**

![proclib_concat](images/proclib_concat.jpg)



## IBMUSER.CNTL

ISPF 3.2 : View properties of **DSNC10.SDSNSAMP**

![sdsnsamp](images/sdsnsamp.jpg)

Allocate **'IBMUSER.CNTL'**

![ibmuser_cntl](images/ibmuser_cntl.jpg)


edit **ADCD.LIB.JCL(INITVOL)**
create **'IBMUSER.CNTL(INITVOL)'**
c99


## IPL and Shutdown Controls

IPL startup procedures defined in **ADCD.Z25A.PARMLIB(VTAMDB)** and **ADCD.Z25A.PARMLIB(VTAMALL)** etc...

```
COMMANDPREFIX=NONE   /* THIS IS THE DEFAULT VALUE */                   
PAUSE 10      /* WAIT FOR SYS TO GET UP FIRST  */                      
S RRS,SUB=MSTR,GNAME=&SYSNAME                                          
S TSO         /* TIME SHARING OPTION */                                
S SDSF                                                                 
S EZAZSSI,P=S0W1                                                       
S TCPIP                                                                
PAUSE 5                                                                
S TN3270                                                               
PAUSE 5                                                                
/* S INETD */                                                          
PAUSE 10                                                               
/*---------------------- DB2 V12 -------------------------------------*
-DBCG START DB2                                                        
PAUSE 10                                                               
S HTTPD1                                                               
S NFSS                                                                 
S CSF                                                                  
S HZSPROC                                                              
PAUSE 5                                                                
S SSHD                                                                 
%CSQ9 START QMGR                                                       
PAUSE 15                                                               
%CSQ9 START CHINIT                                                     
S ASCH,SUB=MSTR                                                        
PAUSE 5                                                                
S JMON                                                                 
PAUSE 3                                                                
S RSED                                                                 
S RSEAPI                                                               
PAUSE 3                                                                
S CFZCIM                                                               
PAUSE 10                                                               
/* S IZUANG1    */   
/* S IZUSVR1    */                                     
D T                          /* DISPLAY ENDING TIME    
* COMMENT LINES MAY ALSO START WITH A SINGLE ASTERISK. 
* REMEMBER - BLANK LINES ARE A BIG NO-NO.              
```

Must Add
```
S ZOSCSRV
S FTPD   
S INETD  
```

![vtamdb](images/vtamdb.jpg)

And for Shutdown, edit **ADCD.Z25A.PARMLIB(SHUTALL)** and **ADCD.Z25A.PARMLIB(SHUTDB)**

Must Add
```
P INETD  
P FTPD   
P ZOSCSRV
```

## Extra Volumes

USERXX volumes - mod 27s - allocate with **smspool.sh** 

```
alcckd /home/ibmsys1/Z25A001/USER0A -d3390-27
alcckd /home/ibmsys1/Z25A001/USER0B -d3390-27
alcckd /home/ibmsys1/Z25A001/USER0C -d3390-27
alcckd /home/ibmsys1/Z25A001/USER0D -d3390-27

```

EAVXXX volumes - mod 9s for SMPE - allocate with **eavpool.sh**

```
alcckd /home/ibmsys1/Z25A001/EAV00A -d3390-9
alcckd /home/ibmsys1/Z25A001/EAV00B -d3390-9
alcckd /home/ibmsys1/Z25A001/EAV00C -d3390-9
alcckd /home/ibmsys1/Z25A001/EAV00D -d3390-9
alcckd /home/ibmsys1/Z25A001/EAV00E -d3390-9
alcckd /home/ibmsys1/Z25A001/EAV00F -d3390-9
```

ZCXXXX volumes - mod 27s - allocate with **zcxpool.sh** 

```
alcckd /home/ibmsys1/Z25A001/ZCX00A -d3390-27
alcckd /home/ibmsys1/Z25A001/ZCX00B -d3390-27
alcckd /home/ibmsys1/Z25A001/ZCX00C -d3390-27
alcckd /home/ibmsys1/Z25A001/ZCX00D -d3390-27
```

Esoteric Devices - mod 27 - allocate with **esopool.sh** 

```
alcckd /home/ibmsys1/Z25A001/WORK01 -d3390-27
```

Edit the **devmapz25a.txt**

```
# Neale Devmap for ZDT V13.0 created 20210303
# Volume directory: /home/ibmsys1/Z25A001

[system]
memory      26000m          # define storage size for virtual host
processors  8               # number of processors
3270port    3270            # port number for TN3270 connections

[manager]
name aws3274 0002           # define a few 3270 terminals
device 0700 3279 3274 mstcon
device 0701 3279 3274 tso1
device 0702 3279 3274 tso2
device 0703 3279 3274 tso3
device 0704 3279 3274 tso4

# [manager]
# name awsrdr 010C           # define a card reader for job submission
# device 00C 2540 2821 /home/ibmsys1/ZDT1204/../cards/*

[manager]  # tap define network adapter (OSA) for communication with Linux
name awsosa 0024 --path=A1 --pathtype=OSD --tunnel_intf=y   # QDIO mode
device 400 osa osa --unitadd=0
device 401 osa osa --unitadd=1
device 402 osa osa --unitadd=2

[manager]  # OSA define OSA for general network communication
name awsosa 0022 --path=F0 --pathtype=OSD 
device 404 osa osa  
device 405 osa osa  
device 406 osa osa  

[manager]
name awsckd 0001
device 0A80 3390 3390 /home/ibmsys1/Z25A001/A5RES1
device 0A81 3390 3390 /home/ibmsys1/Z25A001/A5BLZ1
device 0A82 3390 3390 /home/ibmsys1/Z25A001/A5SYS1
device 0A83 3390 3390 /home/ibmsys1/Z25A001/A5C560
device 0A84 3390 3390 /home/ibmsys1/Z25A001/A5C551
device 0A85 3390 3390 /home/ibmsys1/Z25A001/A5CFG1

device 0A86 3390 3390 /home/ibmsys1/Z25A001/A5DBAR
device 0A87 3390 3390 /home/ibmsys1/Z25A001/A5USS3
device 0A88 3390 3390 /home/ibmsys1/Z25A001/A5PRD5
device 0A89 3390 3390 /home/ibmsys1/Z25A001/A5DBC1
device 0A8A 3390 3390 /home/ibmsys1/Z25A001/A5DBC2

device 0A8B 3390 3390 /home/ibmsys1/Z25A001/A5DIS1
device 0A8C 3390 3390 /home/ibmsys1/Z25A001/A5DIS2
device 0A8D 3390 3390 /home/ibmsys1/Z25A001/A5DIS3

device 0A8F 3390 3390 /home/ibmsys1/Z25A001/A5IMF1
device 0A90 3390 3390 /home/ibmsys1/Z25A001/A5INM1
device 0A91 3390 3390 /home/ibmsys1/Z25A001/A5KAN1

device 0A92 3390 3390 /home/ibmsys1/Z25A001/A5PAGA
device 0A93 3390 3390 /home/ibmsys1/Z25A001/A5PAGB
device 0A94 3390 3390 /home/ibmsys1/Z25A001/A5PAGC

device 0A95 3390 3390 /home/ibmsys1/Z25A001/A5PRD1
device 0A96 3390 3390 /home/ibmsys1/Z25A001/A5PRD2
device 0A97 3390 3390 /home/ibmsys1/Z25A001/A5PRD3
device 0A98 3390 3390 /home/ibmsys1/Z25A001/A5PRD4

device 0A99 3390 3390 /home/ibmsys1/Z25A001/A5RES2
device 0A9A 3390 3390 /home/ibmsys1/Z25A001/A5USR1
device 0A9B 3390 3390 /home/ibmsys1/Z25A001/A5USS1
device 0A9C 3390 3390 /home/ibmsys1/Z25A001/A5USS2
device 0A9D 3390 3390 /home/ibmsys1/Z25A001/A5W901
device 0A9E 3390 3390 /home/ibmsys1/Z25A001/A5W902
device 0A9F 3390 3390 /home/ibmsys1/Z25A001/A5ZWE1 

device 0AA0 3390 3390 /home/ibmsys1/Z25A001/A5PRD1
device 0AA1 3390 3390 /home/ibmsys1/Z25A001/A5PRD2
device 0AA2 3390 3390 /home/ibmsys1/Z25A001/A5PRD3
device 0AA3 3390 3390 /home/ibmsys1/Z25A001/A5PRD4


device 0AA0 3390 3390 /home/ibmsys1/Z25A001/USER0A
device 0AA1 3390 3390 /home/ibmsys1/Z25A001/USER0B
device 0AA2 3390 3390 /home/ibmsys1/Z25A001/USER0C
device 0AA3 3390 3390 /home/ibmsys1/Z25A001/USER0D

device 0AA4 3390 3390 /home/ibmsys1/Z25A001/ZCX00A
device 0AA5 3390 3390 /home/ibmsys1/Z25A001/ZCX00B
device 0AA6 3390 3390 /home/ibmsys1/Z25A001/ZCX00C
device 0AA7 3390 3390 /home/ibmsys1/Z25A001/ZCX00D

device 0AA8 3390 3390 /home/ibmsys1/Z25A001/EAV00A
device 0AA9 3390 3390 /home/ibmsys1/Z25A001/EAV00B
device 0AAA 3390 3390 /home/ibmsys1/Z25A001/EAV00C
device 0AAB 3390 3390 /home/ibmsys1/Z25A001/EAV00D
device 0AAC 3390 3390 /home/ibmsys1/Z25A001/EAV00E
device 0AAD 3390 3390 /home/ibmsys1/Z25A001/EAV00F

device 0600 3390 3390 /home/ibmsys1/Z25A001/WORK01
```

Verify **devmapz25a.txt**

```
[ibmsys1@localhost Z25A001]$ awsckmap devmapz25a.txt
AWSCHK200I Checking DEVMAP file 'devmapz25a.txt' ...
AWSCHK204I Processed 92 records from DEVMAP /home/ibmsys1/Z25A001/devmapz25a.txt
AWSCHK208I Check complete, 0 errors, 0 warnings detected.
```


IPL and Initialise Volumes by editing and submitting **IBMUSER.CNTL(INITVOL)** several times

![initvol](images/initvol.jpg)

from the master console reply **r NN,U**

![initvol_confirm](images/initvol_confirm.jpg)


Add the Esoteric Volume to the configuration

Edit **ADCD.Z25A.PARMLIB(VATLSTDB)**
Edit **ADCD.Z25A.PARMLIB(VATLST00)**

Add Line ```WORK*,0,0,3390     ,Y ```

![VATLSTXX](images/VATLSTXX.jpg)


## Convert Volumes to SMS and enable/disable creation  of new files.

Enable the for read-write only for the duration of SMPE installs


Don't want to accidentally put other shit on these volumes.

```
/* REXX */
cmd.1 = "v sms,volume(EAV00A),enable"
cmd.2 = "v sms,volume(EAV00B),enable"
cmd.3 = "v sms,volume(EAV00C),enable"
cmd.4 = "v sms,volume(EAV00D),enable"
cmd.5 = "v sms,volume(EAV00E),enable"
cmd.6 = "v sms,volume(EAV00F),enable"
cmd.0 = 6
do i = 1 to cmd.0
  x = AXRCMD(cmd.i,var.,5)
  do j = 1 to var.0
      x = AXRWTO(var.j)
  end
end
```

Save above as SYS1.SAXREXEC(EAVON).

```
/* REXX */
cmd.1 = "v sms,volume(EAV00A),disable,new"
cmd.2 = "v sms,volume(EAV00B),disable,new"
cmd.3 = "v sms,volume(EAV00C),disable,new"
cmd.4 = "v sms,volume(EAV00D),disable,new"
cmd.5 = "v sms,volume(EAV00E),disable,new"
cmd.6 = "v sms,volume(EAV00F),disable,new"
cmd.0 = 6
do i = 1 to cmd.0
  x = AXRCMD(cmd.i,var.,5)
  do j = 1 to var.0
      x = AXRWTO(var.j)
  end
end
```

Save above as SYS1.SAXREXEC(EAVOFF)

Master Console invocation
* F AXR,EAVON
* F AXR,EAVOFF
  
  
Convert them to SMS  
```
F AXR,EAVON
	
//CONVERT   EXEC   PGM=ADRDSSU,REGION=0M              
//SYSPRINT  DD     SYSOUT=H                           
//INVOL     DD     VOL=SER=EAV00F,UNIT=3390,DISP=SHR  
//SYSIN     DD *                                      
 CONVERTV -                                           
 DDNAME(INVOL) -                                      
 SMS                                                  
/*
```  

Check the volumes are all available in ISMF

![eavextsg](images/eavextsg.jpg)
  
Create a BIGZFS spanning all 6 volumes

```
//DEFINE    EXEC   PGM=IDCAMS                        
//SYSPRINT  DD     SYSOUT=H                          
//SYSUDUMP  DD     SYSOUT=H                          
//AMSDUMP   DD     SYSOUT=H                          
//SYSIN     DD     *                                 
     DELETE 'SMPEWORK.ZFS'                               
     DEFINE CLUSTER (NAME(SMPEWORK.ZFS) -                
            VOLUMES (EAV00A EAV00B EAV00C -          
                     EAV00D EAV00E EAV00F) -         
            DATACLASS(DCEXTEAV) -                    
            LINEAR CYL(3336 3336) SHAREOPTIONS(3))   
/*                                                   
//FORMAT    EXEC  PGM=IOEAGFMT,REGION=0M,            
// PARM=('-aggregate SMPEWORK.ZFS -compat')              
//SYSPRINT  DD     SYSOUT=H                          
//STDOUT    DD     SYSOUT=H                          
//STDERR    DD     SYSOUT=H                          
//SYSUDUMP  DD     SYSOUT=H                          
//CEEDUMP   DD     SYSOUT=H                          
//*
```	


Mount the Aggregate on /u/ibmuser/smpework
from putty

```
cd /u/ibmuser
mkdir smpework
mount -f SMPEWORK.ZFS /u/ibmuser/smpework
```


For transmission of ShopZ downloads

Turn on Fast TCP on P52

```
su -
/root/fasttcp.sh
```

FTP from Windows ( command line )

```
from Windows ??? navigate to 
S:\ZSHOP\20210921_DVM\7312760424_000010_PROD

pscp ???r * ibmuser@192.168.1.191:/u/ibmuser/STP59536
```
  
  
## Define ALIASes

Keeps the master catalogs small

DEFINE ALIAS (NAME('NEALE') RELATE('USERCAT.Z24C.USER'))  

## ISMF - SMS

M.2
Option 0 - enable storage admin view
exit ISMF

Go back in to check out storage groups SGBASE, SGEXTEAV


## JES2 CHKPT Datasets

Is this still a necessary step in ADCD Z25A ?

