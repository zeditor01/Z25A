# ShopZ Usage.

## ShopZ Access

logon with IBM userid and password.
order Serverpacs ( which will tend to be portable software instances now )
Downloads must be performed using secure methods ( https or ftps )
RACF certificates and keyrings must be used, and they must be valid
z/OSMF must be setup correctly to use certificates

## RACF Certificates

Two certificates
1. Trust ShopZ certificate - to download from Download Server
2. z/OSMF certificate - for z/OSMF to behave securely

### ShopZ Trust

Given a copy by Andrew.

S:\ZSHOP\TLS\digicert.global.root.ca.cert
LABEL 'DIGITAL GLOBAL ROOT CA'

RACDCERT CERTAUTH LIST(SERIALNUMBER(083BE056904246B1A1756AC95991C74A))

RACDCERT CERTAUTH LIST(LABEL('DIGICERT GLOBAL ROOT CA'))


![shopzcert](images/digital_global_root_ca.png)

Tried to Add it, but found it's already there

```
RACDCERT ID(IBMUSER) ADDRING(SHOPZ)
RACDCERT LISTRING(SHOPZ)
RACDCERT ADD('IBMUSER.SHOPZCRT') CERTAUTH TRUST WITHLABEL ('IBMSHOPZ')
RACDCERT ID(IBMUSER) CONNECT(CERTAUTH LABEL('IBMSHOPZ') RING((SHOPZ) USAGE(CERTAUTH) DEFAULT)

RACDCERT ID(IBMUSER) CONNECT(CERTAUTH LABEL('DIGICERT GLOBAL ROOT CA') RING((SHOPZ) USAGE(CERTAUTH) DEFAULT)

```

Not quit sure how the TTLERules later on worked, but I assume this certificate is a SITE CA Trust or something similar.

### z/OSMF Certificate

Stumbled across problems when performing a portable software instance installation, getting errors during z/OSMF workflow. 

```
IZUG1040E A secure connection to system "S0W1" could not be established.
```

Also noticed that the browser was not establishing a secure connection. This was not a problem in the basic dialogs, 
but probably was a problem when the z/OSMF PSI dialog tried to access another zOSMF service. 
z/OSMF ADCD Z25A had an expired certificate for use with ShopZ ... DOH !

```
RACDCERT CERTAUTH LIST(LABEL('zOSMFCA'))
Digital certificate information for CERTAUTH:
  Label: zOSMFCA
  Certificate ID: 2QiJmZmDhZmjganW4tTGw8FA
  Status: TRUST
  Start Date: 2013/11/18 16:00:00
  End Date:   2021/08/21 15:59:59
  Serial Number:
       >00<
  Issuer's Name:
       >CN=z/OSMF CertAuth for Security Domain.OU=IZUDFLT<
  Subject's Name:
       >CN=z/OSMF CertAuth for Security Domain.OU=IZUDFLT<
  Signing Algorithm: sha1RSA
  Key Usage: CERTSIGN
  Key Type: RSA
  Key Size: 1024
  Private Key: YES
  Certificate Fingerprint (SHA256):
       A3:F2:E0:6F:09:94:E4:9D:68:00:48:8F:3F:4B:EF:9F:
       9F:C8:B1:50:0A:de:57:23:AB:DC:B1:02:0B:A4:33:07
  Ring Associations:
    Ring Owner: IZUSVR
    Ring:
       >IZUKeyring.IZUDFLT<
    Ring Owner: START1
    Ring:
       >JES2EDS<
```

Expired certificates can't be renewed / extended. ( you need to do that before expirey ).



Finally remote the "not secure" error in the browser by replacing the IP Address with a hostname .

```
SSL_BAD_CERT_DOMAIN

https://192.168.1.191:10443/zosmf/ 
```


#### Setup new certificate etc...

Step 1 : Generate new self signed CA certificate , certificate & refresh RACF.


```
racdcert certauth gencert subjectsdn(cn('zOSMF CA 2') o('IBM') c('AU')) 
size(2048) withlabel('zOSMFCA2') notafter(date(2031-12-31))

racdcert id(izusvr) gencert subjectsdn(cn('zOSMF Server 2') o('IBM')
c('AU')) size(2048) withlabel('zOSMFServer2') notafter(date(2031-12-31))
signwith(certauth label('zOSMFCA2'))

setr raclist(digtcert) refr
```

Step 2 : Plonk them in a keyring

```
racdcert addring(zOSMFRing2) id(izusvr)

racdcert connect(ring(zOSMFRing2) certauth label('zOSMFCA2')) id(izusvr)

racdcert connect(ring(zOSMFRing2) id(izusvr) label('zOSMFServer2')
default) id(izusvr)

racdcert listring(zOSMFRing2) id(izusvr)
```

```
Digital ring information for user IZUSVR:                             
                                                                      
  Ring:                                                               
       >zOSMFRing2<                                                   
  Certificate Label Name             Cert Owner     USAGE      DEFAULT
  --------------------------------   ------------   --------   -------
  zOSMFCA2                           CERTAUTH       CERTAUTH     NO   
  zOSMFServer2                       ID(IZUSVR)     PERSONAL     YES  
                                                                      
***                                                                   
```

Step 3 : update z/OSMF config to refer to the new keyring

Find out that ZUSVR1 invokes PARMLIB member AS (from SDSF output)

```
**************** TOP OF DATA ****************************
      J E S 2  J O B  L O G  --  S Y S T E M  S 0 W 1  --
                                                         
 ---- SATURDAY,  08 OCT 2022 ----                        
  $HASP373 IZUSVR1  STARTED                              
  IEF403I IZUSVR1 - STARTED - TIME=09.47.08              
  +IEE252I MEMBER IZUPRMAS FOUND IN FEU.Z25A.PARMLIB     
  +CWWKE0001I: The server zosmfServer has been launched. 
```

Edit ADCD.Z25A.PROCLIB(IZUSVR1) to set IZUPRM='AS'

```
//IZUSVR1  PROC PARMS='zosmfServer',     /* Server parms */        
//             ROOT='/usr/lpp/zosmf',    /* zOSMF install root */  
//             WLPDIR='/usr/lpp/zosmf/liberty', /* Liberty DIR */  
//             OUTCLS='*',               /* Sysout class */        
//             USERDIR='/global/zosmf',     /* Config dir */       
//             TRACE='N',                /* Trace option */        
//             KCINDEX='N',            /* KC index rebuild flag */ 
//             IZUPRM='AS',      /* Parmlib suffixes or PREV */    
//             SERVER='AUTOSTART',       /* AUTOSTART server */    
//             Z='0',                    /* Reserved for IBM */    
//             IZUMEM='NOLIMIT'            /* Server memlimit */   
```


Edit IZUPRMAS ( in both ADCD and FEU Proclibs )
* ADCD.Z25A.PARMLIB(IZUPRMAS)
* FEU.Z25A.PARMLIB(IZUPRMAS)

```
KEYRING_NAME('zOSMFRing2')
```

Start z/OSMF with manual override on the PARM member if needed.
```
s izusvr1,izuprm=as
```

these steps, combined with the PAGENT rules below, seemed to get HTTPS transfers from ShopZ to work directly to z/OS




## TLS Configuration.

Setup PAGENT

ADCD.Z25A.PROCLIB(PAGENT)
```
//PAGENT   PROC
//*
//* IBM Communications Server for z/OS
//*
//PAGENT   EXEC PGM=PAGENT,REGION=0M,TIME=NOLIMIT,
//         PARM='ENVAR("_CEE_ENVFILE_S=DD:STDENV")/'
//SYSPRINT DD   SYSOUT=*
//STDENV   DD   *
PAGENT_CONFIG_FILE=/etc/pagent.conf
PAGENT_LOG_FILE=/tmp/pagent.log
LIBPATH=/usr/lib
TZ=AEST-10AEDT
/*
```

/etc/pagent.conf
```
LogLevel 255
TcpImage TCPIP /etc/pagent.TCPIP.conf FLUSH PURGE 600
```

/etc/pagent.TCPIP.conf

```
TTLSConfig /etc/pagent.TTLSRule.policy
```

/etc/pagent.TTLSRule.policy


```
TTLSRule                          shopz
{
RemoteAddrGroupRef              shopzRAG
RemotePortRange                 21
Direction                       Outbound
TTLSGroupActionRef              shopzGA
TTLSEnvironmentActionRef        shopzEA
}
IpAddrGroup                       shopzRAG
{
IpAddr
{
# @host deliverycb-mul.dhe.ibm.com
Addr                          129.35.224.118
}
IpAddr
{
# @host deliverycb-bld.dhe.ibm.com
Addr                          170.225.126.47
}
}
TTLSGroupAction                   shopzGA
{
TTLSEnabled                     On
}
TTLSEnvironmentAction             shopzEA
{
HandshakeRole                   Client
TTLSKeyringParmsRef             shopzKP
TTLSEnvironmentAdvancedParmsRef shopzEAP
}
TTLSEnvironmentAdvancedParms      shopzEAP
{
TLSv1.1                         Off
TLSv1.2                         On
ApplicationControlled           On
SecondaryMap                    On
}
TTLSKeyringParms                  shopzKP
{
Keyring                         IBMUSER/SHOPZ
}
```


Keyring shite that didnt work

```
RACDCERT ID(IBMUSER) ADDRING(SHOPZ)

RACDCERT LISTRING(SHOPZ)

Allocate VB, LRECL84
IBMUSER.SHOPZCRT


RACDCERT ID(IBMUSER) ADDRING(SHOPZ)

RACDCERT LISTRING(SHOPZ)

RACDCERT ADD('IBMUSER.SHOPZCRT') CERTAUTH TRUST WITHLABEL ('IBMSHOPZ')

RACDCERT ID(IBMUSER) CONNECT(CERTAUTH LABEL('IBMSHOPZ') RING((SHOPZ) USAGE(CERTAUTH) DEFAULT)


RACDCERT ID(IBMUSER) LIST(LABEL('IBMSHOPZ'))


RACDCERT ADD('IBMUSER.SHOPZCRT') CERTAUTH TRUST WITHLABEL ('IBMSHOPZ')

IRRD109I The certificate cannot be added.  Profile 083BE056904246B1A1756AC95991
C74A.CN=DigiCert¢Global¢Root¢CA.OU=www.digicert.com.O=DigiCert¢Inc.C=US is alrea
dy defined.
***
```




## ShopZ PSI Download Process.

Order a PSI Serverpac. ( Classic CDC for IMS )

Review the Download Package

![shopz01](images/shopz01.jpg)

Open the Server XML info, and copy the Server XML snippet to the clipboard.

```
=== Order Size and File System Size Information ========================
                                                                        
The size of your order is 549 MB                                        
                                                                        
You need space in the file system used by z/OSMF Software Management    
Add Portable Software Instance for approximately twice the size of your 
order. To convert to 3390 cylinders, multiply the number of MB by 1.25  
and then multiply by 2.                                                 
                                                                        
For example, for a size of 5000 MB, then:                               
( (5,000 MB) * (1.25 CYL/MB) ) * 2 = 12,500 cylinders                   
                                                                        
== Server XML for Add Portable Software Instance From Download Server ==
You can copy the below statements into the z/OSMF Software Management   
Server XML box.                                                         
                                                                        
<SERVER                                                                 
  host="deliverycb-mul.dhe.ibm.com"                                     
  user="P61f4395"                                                       
  pw="b8346803787q36r"                                                  
  >                                                                     
  <PACKAGE                                                              
      file="2022092900018/PROD/content/GIMPAF.XML"                      
      hash="000698051AF35E2A9B5307FD7E65B6CCE7C5542C"                   
      id="ST251564.content"                                             
   >                                                                    
  </PACKAGE>                                                            
</SERVER>      
```

Open z/OSMF, and open Software Configuration.

```
https://192.168.1.191:10443/zosmf/ 
```

Choose "portable software instances"

![psi01](images/psi01.jpg)

Add from Download Server

![psi02](images/psi02.jpg)

Page 1

![psi03](images/psi03.jpg)

Page 2 

![psi04](images/psi04.jpg)

Submit Job

![psi05](images/psi05.jpg)

Let the Software Download Job Run. Watch the smpework ZFS fill up.

```
IBMUSER:/u/ibmuser/smpework/ccdcims: >ls
CPYRIGHT                              S0005.CB.ST251564.CAC.ACACLOAD.pax.Z  S0010.CB.ST251564.CAC.ACACSKEL.pax.Z
GIMPAF.XML                            S0006.CB.ST251564.CAC.ACACMAC.pax.Z   S0011.CB.ST251564.CAC.SCACBASE.pax.Z
IZUD00DF.json                         S0007.CB.ST251564.CAC.ACACMSGS.pax.Z  S0012.CB.ST251564.CAC.SCACCONF.pax.Z
S0003.CB.ST251564.CAC.ACACBASE.pax.Z  S0008.CB.ST251564.CAC.ACACSAMP.pax.Z  S0013.CB.ST251564.CAC.SCACLOAD.pax.Z
S0004.CB.ST251564.CAC.ACACCONF.pax.Z  S0009.CB.ST251564.CAC.ACACSIDE.pax.Z
IBMUSER:/u/ibmuser/smpework/ccdcims: >
```

Back to Step 3 of the Download workflow. Refresh etc... Remember to complete Steps 3 & 4 before going to deployment of PSI.

![psi06](images/psi06.jpg)

You Supplied the Job Card, which allows you to watch progress in SDSF.

![psi07](images/psi07.jpg)

As the download progresses, not that as well as downloading the ST251564 sysmod, it is also downloading all the SMPE work datasets.

```
-rw-rw-rw-   1 OMVSKERN SYS1      225792 Oct  7 20:35 S0114.CB.ST251564.SMPE.GLOBAL.CSI.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1      677376 Oct  7 20:35 S0115.CB.ST251564.SMPE.MVS.DLIB.CSI.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1      806400 Oct  7 20:35 S0116.CB.ST251564.SMPE.MVS.TARGET.CSI.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct  7 20:35 S0117.CB.ST251564.SMPE.MVSD100.SMPDLOG.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct  7 20:35 S0118.CB.ST251564.SMPE.MVSD100.SMPDLOGA.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct  7 20:35 S0119.CB.ST251564.SMPE.MVST100.SMPMTS.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       64512 Oct  7 20:35 S0120.CB.ST251564.SMPE.MVST100.SMPSCDS.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct  7 20:36 S0121.CB.ST251564.SMPE.MVST100.SMPSTS.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct  7 20:36 S0122.CB.ST251564.SMPE.MVST100.SMPTLOG.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct  7 20:36 S0123.CB.ST251564.SMPE.MVST100.SMPTLOGA.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct  7 20:36 S0124.CB.ST251564.SMPE.SMPGLOG.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct  7 20:36 S0125.CB.ST251564.SMPE.SMPGLOGA.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1     14356096 Oct  7 20:38 S0126.CB.ST251564.SMPE.SMPPTS.pax.Z
```


The z/OSMF UI does not notify successful completion of job. Hence, logon to SDSF and wait for RC 0000.



```
1                       J E S 2  J O B  L O G  --  S Y S T E M  S 0 W 1  --  N O D E  S 0 W 1
0
 11.24.43 JOB03190 ---- SATURDAY,  08 OCT 2022 ----
 11.24.43 JOB03190  IRR010I  USERID IBMUSER  IS ASSIGNED TO THIS JOB.
 11.24.43 JOB03190  ICH70001I IBMUSER  LAST ACCESS AT 09:58:21 ON SATURDAY, OCTOBER 8, 2022
 11.24.43 JOB03190  $HASP373 IBMUSERP STARTED - INIT 1    - CLASS A        - SYS S0W1
 11.24.43 JOB03190  IEF403I IBMUSERP - STARTED - TIME=11.24.43
 11.24.44 JOB03190  -                                      -----TIMINGS (MINS.)------                          -----PAGING COUNTS----
 11.24.44 JOB03190  -STEPNAME PROCSTEP    RC   EXCP   CONN       TCB       SRB  CLOCK          SERV  WORKLOAD  PAGE  SWAP   VIO SWAPS
 11.24.44 JOB03190  -PREPODIR             00     96      0       .00       .00     .0             2  BATCH        0     0     0     0
 11.47.44 JOB03190  -GIMGTPKG             00  76451      0       .05       .00   22.9             6  BATCH        0     0     0     0
 11.47.44 JOB03190  IEF404I IBMUSERP - ENDED - TIME=11.47.44
 11.47.44 JOB03190  -IBMUSERP ENDED.  NAME-SHOPZ PSI            TOTAL TCB CPU TIME=      .05 TOTAL ELAPSED TIME=  23.0
 11.47.44 JOB03190  $HASP395 IBMUSERP ENDED - RC=0000
0------ JES2 JOB STATISTICS ------
-  08 OCT 2022 JOB EXECUTION DATE
-           87 CARDS READ
-        2,897 SYSOUT PRINT RECORDS
-            0 SYSOUT PUNCH RECORDS
-          150 SYSOUT SPOOL KBYTES
-        23.00 MINUTES EXECUTION TIME

```


Press next on Page 3

![psi08](images/psi08.jpg)






## ShopZ PSI Install Process.

Open z/OSMF Deployments

![deploy01](images/deploy01.jpg)


Specify New Deployment

![deploy02](images/deploy02.jpg)

Select The Software to Deploy

![deploy03](images/deploy03.jpg)

Choose Classic CDC for IMS - Serverpac ST251564

![deploy04](images/deploy04.jpg)

Deployment Objective is New Global CSI

![deploy05](images/deploy05.jpg)

Check for Missing SYSMODS ( Nah )

![deploy06](images/deploy06.jpg)

Configure This Deployment

![deploy07](images/deploy07.jpg)

welcome - YES 
DLIBS - YES 
Model - Deployment Source 
SMPE Zones - Names from model TGT + DLIB 
Datasets - Names, volumes, storage groups : select ALL ... modify all  - volume = USER0E
Catalogs - CATALOG.Z25A.MASTER 
Volumes and Storage Classes - default 
Mount Points - define the eventual mount point ( but you need to do that outside PSI )  /u/wallen/hooli - CB.OSHOOLI.ZFS 


Zones - are MVST00 and MVSD100
![deploy08](images/deploy08.jpg)

Edit the Target Volumes ( USER0E )
![deploy09](images/deploy09.jpg)

Edit the Target Mount Point 

![deploy10](images/deploy10.jpg)

Define the Job Settings

![deploy11](images/deploy11.jpg)

View the Deployment Summary

![deploy12](images/deploy12.jpg) 

Submit Deployment Jobs

![deploy13](images/deploy13.jpg)

The first Job is (hopefully) avoidable RACF controls that can be ignored.
I just did Override Complete.

Now I submit the second job ( Unzip : IZUD02UZ ).

![deploy14](images/deploy14.jpg)

Now I submit the third job ( Rename : IZUD03RN ).

Now I submit the fourth job ( Update CSI : IZUD04UC ).

And it seems to have worked !

![deploy15](images/deploy15.jpg)


Now take a look at the Workflows

![deploy16](images/deploy16.jpg)


Just Click thru the bullshit stuff

![deploy17](images/deploy17.jpg)

Perform the Post-Install Stuff. This is environment stuff, prior to instance creation.

![deploy18](images/deploy18.jpg)

