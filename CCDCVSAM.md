# CCDCVSAM PSI.


## ShopZ PSI Download Process.

Order a PSI Serverpac. ( Classic CDC for IMS )
Open the Server XML info, and copy the Server XML snippet to the clipboard.

```
=== Order Size and File System Size Information ========================
The size of your order is 549 MB                                        
                                                                        
== Server XML for Add Portable Software Instance From Download Server ==
You can copy the below statements into the z/OSMF Software Management   
Server XML box.                                                         
                                                                        
<SERVER                                                                 
  host="deliverycb-mul.dhe.ibm.com"                                     
  user="P662p007"                                                       
  pw="k6321212q47798D"                                                  
  >                                                                     
  <PACKAGE                                                              
      file="2022101100016/PROD/content/GIMPAF.XML"                      
      hash="B464A6FF704B1718E703C28DCD3ADFFDD5E14214"                   
      id="ST251603.content"                                             
   >                                                                    
  </PACKAGE>                                                            
</SERVER>                                                               ```

Open z/OSMF, and open Software Configuration.

```
https://192.168.1.191:10443/zosmf/ 
```

Choose "portable software instances".

Page 1
* name = CCDCVSAM 
* SERVER = paste XML
* System = S0W1
* USS Directory = /u/ibmuser/smpework/cdcv

Page 2. Accept previous defaults ( CLIENT XML and JOB Card ) 

Submit Job

Watch the smpework ZFS fill up.

```
IBMUSER:/u/ibmuser/smpework/ccdcvsam: >ls -al
total 1125104
drwxrwxr-x   2 OMVSKERN SYS1        8192 Oct 10 03:53 .
drwxrwxrwx   7 OMVSKERN SYS1        8192 Oct 10 03:21 ..
-rw-rw-rw-   1 OMVSKERN SYS1         405 Oct 10 03:22 CPYRIGHT
-rw-rw-rw-   1 OMVSKERN SYS1       72160 Oct 10 03:22 GIMPAF.XML
-rw-rw-rw-   1 OMVSKERN SYS1        5040 Oct 10 03:53 GIMPAF.XSL
-rw-rw-rw-   1 OMVSKERN SYS1      134013 Oct 10 03:22 IZUD00DF.json
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct 10 03:22 S0003.CB.ST251561.CAC.ACACBASE.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct 10 03:22 S0004.CB.ST251561.CAC.ACACCONF.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1     8870400 Oct 10 03:22 S0005.CB.ST251561.CAC.ACACLOAD.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct 10 03:22 S0006.CB.ST251561.CAC.ACACMAC.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1      387072 Oct 10 03:23 S0007.CB.ST251561.CAC.ACACMSGS.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1      774144 Oct 10 03:23 S0008.CB.ST251561.CAC.ACACSAMP.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct 10 03:23 S0009.CB.ST251561.CAC.ACACSIDE.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       64512 Oct 10 03:23 S0010.CB.ST251561.CAC.ACACSKEL.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct 10 03:23 S0011.CB.ST251561.CAC.SCACBASE.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct 10 03:23 S0012.CB.ST251561.CAC.SCACCONF.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1     10644480 Oct 10 03:24 S0013.CB.ST251561.CAC.SCACLOAD.pax.Z
-rw-rw-rw-   1 OMVSKERN SYS1       32256 Oct 10 03:24 S0014.CB.ST251561.CAC.SCACMAC.pax.Z
```

When Download finishes RC00

Page 3. Accept Job Settings, Job Card etc... for the SMPE jobs to be generated

Page 4. Finish

## PSI Deployment

Follow the normal Deployment workflow
  
* welcome - press next
* DLIBS - YES, we do want to copy the DLIBs
* Model - Accept the Deployment Source as the model
* SMPE Zones - Accept names from model MVST100 & MVSD100
* Datasets - Names, volumes, storage groups : select ALL ... HLQ = CCDC ; Volume = USER0A
* Catalogs - Accept CATALOG.Z25A.MASTER
* Volumes and Storage Classes - default
* Mount Points - /usr/lpp/mqm/V8R0M0	CCDC.OMVS.V8R0M0.MQROOT  (screenshot below)

Datasets

![ccdc03](images/ccdc03.JPG) 

ZFS File Systems

![ccdc04](images/ccdc04.JPG)


Job Settings for the deployment jobs:

```
IBMUSER.DM.D221010.T164457.CNTL
```

Now, Submit the Jobs, one by one, through the z/OSMF user interface.

![ccdc05](images/ccdc05.JPG)

Three jobs are generated.
* IZUD01UZ	Unzip Data Sets: Run This
* IZUD02RN	Rename Data Sets: Run This
* IZUD03UC	Update CSI Data Sets: Run This  

The Unzip job produces this outcome.

![ccdc06](images/ccdc06.JPG)

The rename job produces this outcome.

![ccdc07](images/ccdc07.JPG)

And the CSI Job just updates the information in the CSI zones to reflect what has been done.
The deployment jobs should now all show as complete.

![ccdc08](images/ccdc08.JPG)


## Perform Post-Install Workflows

There are two workflows to review.

![ccdc09](images/ccdc09.JPG)


The "Your Order" Workflow is motherhood and apple pie. Just click yes, yes, yes


![ccdc10](images/ccdc10.JPG)

The Post Deploy Workflow is important for the product to work.

![ccdc11](images/ccdc11.JPG)

Option to Read variables from a properties file of a previous installation ( or not ).

![ccdc12](images/ccdc12.JPG)

RACF Job doing a lot of stuff that I don't understand.

![ccdc13](images/ccdc13.JPG)

Runs fine, RC00, with lots of IBMUSER already has this power informational messages.

![ccdc14](images/ccdc14.JPG)

Job to update a subsystem PARMLIB.

![ccdc15](images/ccdc15.JPG) 

Job to mount the filesystems for MQ.

![ccdc16](images/ccdc16.JPG)

Job to Update the new SMPE DDDEFs.

![ccdc17](images/ccdc17.JPG)

Job to Relink load modeules with CALLLIBs.

![ccdc18](images/ccdc18.JPG)

Job to Write Variables to a property file.

![ccdc19](images/ccdc19.JPG)




