# Steps to deploy eupf based free5gc core


<details><summary>Instruction to deploy as docker-compose</summary>
<p>

### Deploy as docker-compose
Prerequisites

As metioned in free5gc::docker-compose repo:
- Prepared GTP5G kernel module needed to run the UPF
### Compile
```
sudo apt update && sudo apt install make
sudo apt install gcc
cd gtp5g
make clean && make
```

### Install kernel module
Install the module to the system and load automatically at boot
```
sudo make install

```

### Install Docker and Docker-Compose
```
cd
./ready -d -c

```

Actually we use vanilla free5gc::docker-compose with some overrides(see docker-compose.override.yml):
- free5gc-upf service is disabled
- edgecom-upf service is added
- edgecom-nat service is added
- grafana & prometheus services are added

So you can clone free5gc docker-compose and test free5gc upf, then add edgecom override files and feel the differences.


0. Run containers based on docker hub images:
   ```bash
   cd docker-compose
   docker compose pull
   docker compose up -d
   ```
### To undeploy everything
   ```
   docker compose rm

   ```
# After the above deployment 
The following containers should be running

# Next we need to create a subsriber in free5gc webUI

user: admin
password: free5gc
make sure the imsi number in subscriber and in ue_config.yaml file are same 
## PLMN id: 00101
## mcc : 001
## mnc : 01

Just change the PLMN id in free5gc webui




</p>
</details>

# steps to run UERANSIM on host
<details><summary>Instruction to deploy UERANSIM on  host</summary>
<p>
### We can now stop the ueransim container
  
```
docker stop ueransim_container_id
```

## Installation of ueransim
On our vm we install ueransim 

``` 
cd ~
git clone https://github.com/aligungr/UERANSIM
```
Dependencies to install ueransim
```
sudo apt install make
sudo apt install gcc
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo snap install cmake --classic
```
To build ueransim
```
cd ~/UERANSIM
make
```
And that's it. After successfully compiling the project, output binaries will be copied to ~/UERANSIM/build folder. And you should see the following files

    nr-gnb | Main executable for 5G gNB (RAN)
    nr-ue | Main executable for 5G UE
    nr-cli | CLI tool for 5G gNB and UE
    nr-binder | A tool for utilizing UE's internet connectivity.
    libdevbnd.so | A dynamic library for nr-binder

### Steps make nr-ue,nr-gnb,nr-cli,nr-binder as commands

```
cd ~/UERANSIM/build 
```
```
sudo mv nr-ue /usr/bin
sudo mv nr-gnb /usr/bin
sudo mv nr-cli /usr/bin
sudo mv nr-binder /usr/bin
```

## Create the config files for ue and gnb


#Establishing PDU session
After making the config file open different terminals of the vm and 
### On terminal 1 Run:
```
cd ~/UERANSIM/config
nr-gnb -c gnb_config.yaml

```
On success it should show something like:
```
2023-09-17 11:48:43.638] [sctp] [info] Trying to establish SCTP connection... (10.100.200.11:38412)
[2023-09-17 11:48:43.640] [sctp] [info] SCTP connection established (10.100.200.11:38412)
[2023-09-17 11:48:43.640] [sctp] [debug] SCTP association setup ascId[33]
[2023-09-17 11:48:43.640] [ngap] [debug] Sending NG Setup Request
[2023-09-17 11:48:43.642] [ngap] [debug] NG Setup Response received
[2023-09-17 11:48:43.642] [ngap] [info] NG Setup procedure is successful
```
Now after this 
### On terminal 2 Run:
```
cd ~/UERANSIM/config
nr-ue -c ue_config.yaml
```

On success it should show 
```
2023-09-17 11:53:26.495] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-09-17 11:53:26.495] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-09-17 11:53:26.496] [nas] [info] Selected plmn[208/93]
[2023-09-17 11:53:26.496] [rrc] [info] Selected cell plmn[208/93] tac[1] category[SUITABLE]
[2023-09-17 11:53:26.496] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-09-17 11:53:26.496] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-09-17 11:53:26.496] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-09-17 11:53:26.496] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-09-17 11:53:26.496] [nas] [debug] Sending Initial Registration
[2023-09-17 11:53:26.496] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-09-17 11:53:26.496] [rrc] [debug] Sending RRC Setup Request
[2023-09-17 11:53:26.496] [rrc] [info] RRC connection established
[2023-09-17 11:53:26.496] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-09-17 11:53:26.496] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-09-17 11:53:26.513] [nas] [debug] Authentication Request received
[2023-09-17 11:53:26.520] [nas] [debug] Security Mode Command received
[2023-09-17 11:53:26.520] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-09-17 11:53:26.528] [nas] [debug] Registration accept received
[2023-09-17 11:53:26.529] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-09-17 11:53:26.529] [nas] [debug] Sending Registration Complete
[2023-09-17 11:53:26.529] [nas] [info] Initial Registration is successful
[2023-09-17 11:53:26.529] [nas] [debug] Sending PDU Session Establishment Request
[2023-09-17 11:53:26.529] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-09-17 11:53:26.774] [nas] [debug] PDU Session Establishment Accept received
[2023-09-17 11:53:26.774] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-09-17 11:53:26.785] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.60.0.16] is up.

```

Now after uesimtun0 interface is created,we need to ping to check connectivity
```
ping -I uesimtun0 google.com
```


</p>
</details>



