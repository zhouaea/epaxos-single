## Testing Epaxos (NP) and Paxos (NP) in Cloudlab
### Experiment Setup
1. 3 server machines
2. 1 client machine

### Switching Between Paxos (NP) and EPaxos (NP)
#### Paxos
Do `git checkout paxos-no-pipelining-no-batching` to get the setup we used to get the results for Paxos (NP) in Table 1. Then, follow the instructions down below.
#### EPaxos
Do `git checkout epaxos-no-pipelining-no-batching` to get the setup we used to get the results for EPaxos (NP) in Table 1. Then, follow the instructions down below.

### Installation (***For Each Machine***)
1. Make sure Rabia is properly installed. Follow the instructions in the repo. This step is critical as it provides the go binary and python3.8 needed for testing.
2. SSH into each of the VMs and do the following inside `~/go/src`:
    1. ```git clone https://github.com/zhouaea/epaxos-single.git && cd epaxos-single```
    4. ```. compilePaxos.sh``` or ```. compileEPaxos.sh```
        5. This will not output anything if successful.


### Configure Machines

#### Configure the Master   
1. Choose one of your server machines to be the master server.
1. Find the [experiment network ip address](https://docs.cloudlab.us/cloudlab-manual.html#%28part._.Topology_.View%29) of the desired node in your rspec manifest. 
    1. First, find the description of your machine by looking for a `<node>` tag with the attribute `client_id=<your_machine_name>`.
    2. Inside the node tag, look for `<interface>`, and inside it you should see an `<ip/>` tag. The attribute `address` will give you the machine's experiment network ip.
        2. The address likely has the format `10.10.1.x`.
    3. ![Identifying Master Server IP Screenshot](./README-images/Identifying%20Master%20Server%20IP.png)
   
2. Inside the ssh shell of your server machine, using a commandline text editor of your choice, edit `runMasterServer.sh`.
    1. Set `MASTER_SERVER_IP` equal to the server machine's experiment network ip address.

#### Configure the Other Replicas 
1. For every other server machine: 
    1. Find its experiment network ip address.
    2. Inside the ssh shell of the server machine, using a commandline text editor of your choice, edit `runServer.sh`.
        1. Set `MASTER_SERVER_IP` equal to the master server machine's experiment network ip address.
        2. Set `REPLICA_SERVER_IP` to the current server machine's experiment network ip address.

#### Configure the Client
1. Inside the ssh shell of the client machine, using a commandline text editor of your choice, edit `execClient.sh`.
    1. Set `MASTER_SERVER_IP` equal to the master server machine's experiment network ip address.


### Run
1. In your master server machine, run `. runMasterServer.sh`
2. In your replica server machines, run `. runServer.sh`. 
   2. Once all servers are connected,  you should see messages from `rpc.Register` and the message `Replica id: x. Done connecting to peers`
   3. Alternatively, you may see the error below. If this is the case, you did not input the network ip addresses correctly in one or more machines. Type `. fastkill` in each server machine to terminate server processes and retry the configuration steps above.
    ```panic: runtime error: invalid memory address or nil pointer dereference
    [signal SIGSEGV: segmentation violation code=0x1 addr=0x18 pc=0x560ac5]
    
    goroutine 56 [running]:
    genericsmr.(*Replica).waitForPeerConnections(0xc00027c000, 0xc000096480)
            /root/go/src/epaxos-single/src/genericsmr/genericsmr.go:196 +0x105
    created by genericsmr.(*Replica).ConnectToPeers
            /root/go/src/epaxos-single/src/genericsmr/genericsmr.go:127 +0x9f
    ```
2. Finally, run `. execClient.sh > run.txt` in the terminal of your client vm.
    2. After a couple of seconds, press the enter key a few times on your keyboard. You should see a bunch of DONE messages, equal to the number of NClients specified. You can also use the `ps` command to check for pending clients.
    3. Verify the contents of run.txt. It should contain a list of start and end times.
    4. You'll know if `runClient.sh` failed if each client in `NClients` outputs `Error connecting to replica 0` after a timeout period of around 1 minute.


### Analyze
All analysis is done on the client machine.

#### Dependencies

We use `python3.8` for our analysis scripts, which should have been downloaded when you installed Rabia.

Note that our latency analysis script requires the numpy package, which you can download using pip. Both pip and numpy should have been downloaded when you installed Rabia. We have also provided a shell script that will install pip and numpy for python3.8 that can be run with `. download_numpy.py`.

#### Latency and Throughput Analysis
In your client machine, run `. calculate_throughput_latency.sh`. This will run both `latency_analysis.py` and `throughput_analysis.py` using `python3.8`.

You may get an error from `calculate_throughput_latency.sh` if you run it before `runClient.sh` has terminated, or if `runClient.sh` failed. 

#### Latency analysis
In your client machine, run `python3.8 latency_analysis.py`. This will read from the `log.out` file generated by `execClient.sh` and print results into the shell.

If you see `SyntaxError: Non-ASCII character '\xc2' in file latency_analysis.py on line 21, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details`
it's likely because you're using python 2 instead of python 3. 
#### Throughput analysis
In your client machine, run `python3.8 throughput_analysis.py`. This will read the configuration specified in `execClient.sh` and the `run.txt` file you redirected the output of `execClient.sh` into, printing results into the shell.
