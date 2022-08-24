# DB2 zOS Setup

## Db2 CLP

```
Database 2 entry:

 Database alias                       = DALLASC
 Database name                        = DALLASC
 Node name                            = P52DBCG
 Database release level               = 15.00
 Comment                              =
 Directory entry type                 = Remote
 Catalog database partition number    = -1
 Alternate server hostname            =
 Alternate server port number         =
 
Node 5 entry:

 Node name                      = P52DBCG
 Comment                        =
 Directory entry type           = LOCAL
 Protocol                       = TCPIP
 Hostname                       = 192.168.1.191
 Service name                   = 5040
 DCS 2 entry:

 Local database name                = DALLASC
 Target database name               = DALLASC
 Application requestor name         =
 DCS parameters                     =
 Comment                            =
 DCS directory release level        = 0x0100
 
 C:\Program Files\IBM\SQLLIB\BIN>db2 connect to dallasc user ibmuser using sys1

   Database Connection Information

 Database server        = DB2 z/OS 12.1.5
 SQL authorization ID   = IBMUSER
 Local database alias   = DALLASC
 
 ```
 
 ## DDF REST
 
DSNTIJRS
 
RDEFINE DSNR DBCG.REST OWNER(IBMUSER) UACC(NONE)

PERMIT DBCG.REST CLASS(DSNR) ID(FRED) ACCESS(READ)
 
SETROPTS RACLIST(DSNR) REFRESH


 
 DSNTIJR2
 (versioning support)
 
 SYSIBM.DSNSERVICE 
 
 ```
 curl --location --request GET 'http://192.168.1.191:5040/services' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic SUJNVVNFUjpTWVMx'

{
    "DB2Services": [
        {
            "serviceName": "DB2ServiceDiscover",
            "serviceCollectionID": null,
            "version": null,
            "isDefaultVersion": true,
            "serviceStatus": "started",
            "serviceDescription": "DB2 service to list all available services.",
            "serviceProvider": "db2service-1.0",
            "serviceURL": "http://192.168.1.191:5040/services/DB2ServiceDiscover"
        },
        {
            "serviceName": "DB2ServiceManager",
            "serviceCollectionID": null,
            "version": null,
            "isDefaultVersion": true,
            "serviceStatus": "started",
            "serviceDescription": "DB2 service to create, drop, or alter a user defined service.",
            "serviceProvider": "db2service-1.0",
            "serviceURL": "http://192.168.1.191:5040/services/DB2ServiceManager"
        }
    ]
}
```


 
 
 
