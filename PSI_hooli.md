# Using z/OSMF to install a portable software instance from ShopZ

This is the hooli test sample.

1. Order the PSI Serverpac
2. Define the PSI and Download the Serverpac from ShopZ Download Server to ZFS
3. Invoke the z/OSMF Deployment dialog to deploy the hooli PSI

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
  
Next ... Fill in Page 2 details, including the <CLIENT> XML and a Job Card
  
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
  
  

## 3. Invoke the z/OSMF dialog to deploy the hooli PSI



