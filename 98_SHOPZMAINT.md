# Ordering PTFs.

## Case in Point 
z/OSMF Error when performing a portable software instance deployment.

Running Thru PSI Receive. 
* Did Step 1 & 2. 
* Step 3: Submitted https receive Job ( RC00 ). 
* Press Next & Get ISUD277E

```
An error was found in file "/u/ibmuser/smpework/ccdcims/IZUD00DF.json". Error: "The file contains data that is not supported by the current level of z/OSMF.  The version = 8."
```

Known Error - Needs APAR PH42028 ; PTF UI79663 

# Process

Identify the Global Zone for zOSMF = MVS.GLOBAL.CSI 


