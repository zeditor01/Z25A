# Using z/OSMF to install a portable software instance from ShopZ

This is the hooli test sample.

1. Order the PSI Serverpac
2. Define the PSI and Download the Serverpac from ShopZ Download Server to ZFS
3. Invoke the z/OSMF Deployment dialog to deploy the hooli PSI

There is a series of videos covering the same material with detailed verbal explanation. 
The notes below are a scrollable summary for my own benefit.

https://mediacenter.ibm.com/playlist/dedicated/101043781/1_mzzf31vy/1_f1qhec1j


## 1. Order a PSI Serverpac. ( Classic CDC for IMS )

Review the Download Package

![shopz01](images/shopz01.jpg)

Open the Server XML info, and copy the Server XML snippet to the clipboard.

```
<SERVER
host="deliverycb-bld.dhe.ibm.com"
user="B0000000"
pw="123"
>
<PACKAGE
file="zosmf_pswi/GIMPAF.XML"
hash="4BDFCF518AB26E5DB3CB9F29F5505E3B2F62C28D"
id="pswitest.content"
>
</PACKAGE>
</SERVER>
```

and the CLIENT bits

```
<CLIENT
downloadmethod="https"
downloadkeyring="javatruststore"
>
</CLIENT>
```


## 2. Define the PSI and Download the Serverpac from ShopZ Download Server to ZFS 

Open z/OSMF, and open Software Configuration.

```
https://192.168.1.191:10443/zosmf/ 
```

Choose "portable software instances"

![psi01](images/psi01.jpg)

Add from Download Server

![psi02](images/psi02.jpg)

Fill in Page 1 details, including the <SERVER> XML
  
![hooli01](images/hooli01.JPG)
  
Next ... Fill in Page 2 details, including the <CLIENT> XML and a Job Card. Be careful to update the USS directory to what you want.
  
![hooli02](images/hooli02.JPG)

```
//IBMUSERJ JOB  (PSI),'SHOPZ JCL',CLASS=A,MSGCLASS=H,  
//             NOTIFY=&SYSUID,MSGLEVEL=(1,1),REGION=0M   
```
  
Next ... See the Download job generated in Page 3.
  
![hooli03](images/hooli03.JPG)
  
and verify it's location in ISPF  
  
![hooli04](images/hooli04.JPG)  

You should submit the job via z/OSMF workload, so that it can track the end to end progress.  

![hooli05](images/hooli05.JPG)    

Monitor the progress through z/OSMF ( refresh button )
  
![hooli06](images/hooli06.JPG)    

View the Job progress in SDSF

![hooli07](images/hooli07.JPG)    

And verify the download to ZFS via a terminal session
  
![hooli08](images/hooli08.JPG)  

And on Page 4 press Finish
  
![hooli09](images/hooli09.JPG)    
  

## 3. Invoke the z/OSMF dialog to deploy the hooli PSI

Now that the sucker is downloaded to a local ZFS, we need to complete various workflows. 
This is basically a browser user interface to control the SMPE processes that you would otherwise do with JCL, TSO, SMPE and ISPF.
  
Open the z/OSMF Deployments Page, and Start a New Deployment
  
![deployhooli01](images/deployhooli01.JPG)     

You will see a worklow of tasks to perform through this z/OSMF workflow
  
![deployhooli02](images/deployhooli02.JPG)    
 
  	
### 3.1 Specify the properties for this deployment.

Give the deployment a name
  
![deployhooli03](images/deployhooli03.JPG)      
  
### 3.2 Select the software to deploy.

Choose the PSI you want to deploy
  
![deployhooli04](images/deployhooli04.JPG)       
  
### 3.3 Select the objective for this deployment.

Tell it to create a new SMPE CSI (or place the PSI in an existing zone)
  
![deployhooli05](images/deployhooli05.JPG)   
  
### 3.4 Check for Missing SYSMODs.

Optionally check for missing SYSMODs
  
![deployhooli06](images/deployhooli06.JPG)   

### 3.5 Configure this deployment.

This is the SMPE stuff, all wrapped up in a browser dialog, which will be used to create a json file in the USS path. 
This json file contains a structured specification of all the steps, the DDDEFs the volumes or storage groups etc...  
  
![deployhooli07](images/deployhooli07.JPG)     

Won't bother to screenshot every step, but the following notes cover the entries in the different pages
  
* welcome - YES
* DLIBS - YES
* Model - Deployment Source
* SMPE Zones - Names from model TGT + DLIB
* Datasets - Names, volumes, storage groups : select ALL ... modify all  - volume = USER0E
* Catalogs - CATALOG.Z25A.MASTER
* Volumes and Storage Classes - default
* Mount Points - define the eventual mount point ( but you need to do that outside PSI )  /u/wallen/hooli - CB.OSHOOLI.ZFS  
  
  
### 3.5 Define the job settings. z/OSMF creates the deployment summary and jobs.

  
#### 3.6 View the deployment summary.

### 3.7 Submit deployment jobs.

### 3.8 Specify the properties for the target software instance.

