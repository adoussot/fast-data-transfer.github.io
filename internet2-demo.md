[[Home](index.md)]  [[Documentation](doc-fdt-ddcopy.md)]  [[Performance Tests](perf-disk-to-disk.md)] [Internet2 Demo]



### Internet2 2017 Technology Exchange: The Fast Data Transfer Tool
#### Overcoming Limitations to High Performance Transfers Over the Wide Area Network


**Access to your Google Cloud VMs**

Open two windows and ssh into the VMs using the username `fdt` and the IPs and password given at the tutorial.

```
ssh -l fdt <VM1-IP>
ssh -l fdt <VM2-IP>
```

On VM1:
```
export SERVER1=$(hostname --ip-address)
```

On VM2:
```
export SERVER2=$(hostname --ip-address)
```

Export these values on the other VM too, i.e. copy SERVER1 on VMS2 and SERVER2 on VM1. These will be the private IP addresses, not the ones you used above for login into the VMs.


**Setup and FDT Installation**

On Both VMs:
```
# install java and FDT
sudo yum install -y wget

sudo yum install -y java-1.8.0-openjdk.x86_64
java -version

wget https://github.com/fast-data-transfer/fdt/releases/download/0.26/fdt.jar
java -jar fdt.jar -version
```


**Copy a file**


Send one file called "local.data" from the local system directory to another computer in the "/tmp" folder, with default parameters

First the FDT server needs to be started on the "remote" system (VM2). The default settings will be used, which implies the default port, 54321, on both the client and the server. The -S flag is used to disable the standalone mode, which means that the server will stop after the session will finish.

On VM2:
```
[remote computer]$ java -jar fdt.jar -S
```

Then the client will be started on the "local" system (VM1) specifying the sourcefile, the remote address (or hostname) where the server was started in the previous step and the destination directory.

On VM1:
```
# create a dummy file for the transfer
for i in `seq 100`; do echo "local data"; done > local.data
[local computer]$ java -jar fdt.jar -c $SERVER2 -d /tmp ./local.data
```

_Secure Copy (SCP) Mode_

In this mode the server will be started on the remote systemautomatically by the local FDT client using SSH.

On VM1:
```
[local computer]$ java -jar fdt.jar ./local.data fdt@$SERVER2:/home/fdt/
```

If the remoteuser parameter is not specified the local user, running the fdt command, will beused to login on the remote system.

**Recursive copying**

To get the content of an entire folder and all its children, located in the user's home directory, we will use the `-r` (recursive mode) flag. Furthermore, the `-pull` flag will be used to sink the data from the server. In the client-server mode, the acces to the server will be restricted to local IP addresses only. This is done with the `-f` flag.

Multiple IP addresses may be specified using the -f flag using ':' separator. If the IP address of the client is not specified in the allowed IP addresses, the connection will be closed. In the following command the server is started in standalone mode, which means that will continue to run after the session will finish. The transfer rate for every client session will be limited to 4 MBytes/s.

On VM2:
```
[remote computer]$ java -jar fdt.jar -f $SERVER1:$SERVER2 --limit 4M
```


The command for the local client will be.

On VM1
```
[local computer]$ java -jar fdt.jar -pull -r -c $SERVER2 -d ./share /usr/share  
```

_Recursive copying in SCP mode_

In this mode only the order of the parameters will be changed, and `-r` is the only argument that must be added (`-pull` is implicit). The same authentication policies apply.

On VM1:
```
[local computer]$ java -jar fdt.jar -r  fdt@$SERVER2:/usr/share ./share
```

**Transfer with list of files**

The user can define a list of files (one filename per lin ) to be transferred. FDT will detect if the files are located on multiple devices and will use a dedicated thread for each device.

On VM2
```
[remote computer]$ java -jar fdt.jar -S
```

ON VM1
```
[local computer]$ java -jar fdt.jar -fl ./file_list.txt -c <remote_address> -d /home/fdt/files
```


**Testing network connectivity**

To test the network connectivity one can start a transfer of data from /dev/zero on the server (VM2) to /dev/null on the client (VM1) using 10 streams in blocking mode, for both the server and the client with 8 MBytes buffers. The server will stop after the test is finished 

On the VM2 (server):
```
[remote computer]$ java -jar fdt.jar -bio -bs 8M -f $SERVER1 -S
```

On the VM1 (client):
```
[local computer]$ java -jar fdt.jar -c $SERVER2 -bio -P 10 -d /dev/null /dev/zero
```

 _SCP mode_

On VM1:
```
[local computer]$ java -jar fdt.jar -bio -P 10 /dev/zero $SERVER2:/dev/null
```


**Testing local read/write performance**

To test the local read/write performance of the local disk the
DDCopy may be used.

- The following command will copy the entire partition
/dev/dsk/c0d1p1 to /dev/null reporting every 2 seconds ( the default )
the I/O speed

```
[local computer]$ java -cp fdt.jar lia.util.net.common.DDCopy if=/dev/dsk/c0d1p1 of=/dev/null
```

- To test the write speed of the file system using a 1GB file
read from /dev/zero the following command may be used. The operating
system will sync() the data to the disk. The data will be read/write
using 10MB buffers

```
[local computer]$ java -cp fdt.jar lia.util.net.common.DDCopy  if=/dev/zero of=/home/user/1GBTestFile bs=10M count=100 flags=NOSYNC
```

OR

```
[local computer]$ java -cp fdt.jar lia.util.net.common.DDCopy  if=/dev/zero of=/home/user/1GBTestFile bs=1M bn=10 count=100 flags=NOSYNC
```

**Launching FDT as Agent**

```
java -jar fdt.jar -tp <transfer,ports,separated,by,comma> -p <portNo> -agent
```

- Sending coordinator message to the agent:

```
java -jar fdt.jar -dIP <destination-ip> -dp <destination-port> -sIP <source-ip> -p <source-port> -d /tmp/destination/files -fl /tmp/file-list-on-source.txt -coord
```
- Retrieving session log file. 

To retrieve session log file user needs to provide at least these parameters:

```
java -jar fdt.jar  -c <source-host> -d /tmp/destination/files -sID <session-ID>
```

- To retrieve list of files on custom path there is a custom mode which can be used.

```
java -jar fdt.jar  -c <source-host> -ls /tmp/
```

