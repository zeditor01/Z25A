# Network Customisation

P52 MAC
E8:6A:64:5D:2E:8A
Prev Address Resvn 192.168.1.191 ( for z/OS )

## Home Network Diagram

![n01](images/network01.png)

## Find_IO

   
FIND_IO for "ibmsys1@localhost.localdomain" 

```                                                                                                
         Interface         Current          MAC                IPv4              IPv6           
 Path    Name              State            Address            Address           Address        
------   ----------------  ---------------- -----------------  ----------------  -------------- 
  F0     enp0s31f6         UP, RUNNING      e8:6a:64:5d:2e:8a  192.168.1.188     fe80::8124:21fd:e178:6d63%enp0s31f6  
  F1     wlp0s20f3         DOWN             fa:f4:36:d4:cd:1d  *                 *               
. 
  *      virbr0            UP, NOT-RUNNING  52:54:00:a4:96:d1  192.168.122.1     *               
  *      virbr0-nic        DOWN             52:54:00:a4:96:d1  *                 *               
. 
  A0     tap0              DOWN             02:a0:a0:a0:a0:a0  *                 *               
  A1     tap1              DOWN             02:a1:a1:a1:a1:a1  *                 *               
  A2     tap2              DOWN             02:a2:a2:a2:a2:a2  *                 *               
  A3     tap3              DOWN             02:a3:a3:a3:a3:a3  *                 *               
  A4     tap4              DOWN             02:a4:a4:a4:a4:a4  *                 *               
  A5     tap5              DOWN             02:a5:a5:a5:a5:a5  *                 *               
  A6     tap6              DOWN             02:a6:a6:a6:a6:a6  *                 *               
  A7     tap7              DOWN             02:a7:a7:a7:a7:a7  *                 *
```  
  
## Edit devmapz25a.txt

```
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
```

The --tunnel_ip and --tunnel_mask defaults are as follows: 

![masks](images/masks.jpg)


## cc

![n02](images/network02.png)


![n03](images/network03.png)


![n04](images/network04.png)


![n05](images/network05.png)


![n06](images/network06.png)


![n07](images/network07.png)


![n08](images/network08.png)


![n08](images/network09.png)


![n10](images/network10.png)

