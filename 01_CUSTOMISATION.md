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



