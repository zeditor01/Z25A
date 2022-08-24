# DB2 zOS Setup

Db2 CLP

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
 
