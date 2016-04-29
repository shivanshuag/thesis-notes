# Possible graphs to show

* A graph of when etcd was updated due to memory change along with graph of host used memory.
* A graph of when etcd was updated due to cpu change along with graph of host cpu usage.

* A graph of when a guest was migrated out of a host due to CPU imbalance along with graph of cpu usage.
* A graph of when a guest was migrated out of a host due to memory imbalance along with graph of memory usage.

* Show a graph of idle memory vs memory usage. When used memory goes to 80% of total, idle memory decreases.
*


# First

* Done Without ballooning or migration
* On compute 2, created 4 large VMs and 3 small VMs
* Small VM runs one of the three workloads(it randomly chooses the workload to be run) and reports time taken
* Large VM runs two threads. Each thread runs one of the three workloads and reports time taken.
* Check the usage patterns from 4pm 26-04-2016 to 8am 27-04-2016.

# Second

* Done with ballooning but without migration
* On compute 2, created 4 large VMs and 3 small VMs
* Additionally added a timeout of 150 seconds after every workload run to minimize resource usage
* Check the usage patterns from 8pm 27-04-2016 onwards.
* Changed the sleep interval between two runs to be random number between 0 and 90 at around 12:30 am on 26th April.
* Stopped all the VMs at 13:25 on 28 April.

# Third

* Same setting as experiment 2.
* More logs like when etcd was updated and when migration would occur.
* Started at 15:24 on 28th
* Ended at 11:42 on 29th

# Fourth
* Same setting as experiment 1
* started on compute 3 at 18:23 on 28th April
* Aim to show benefits of ballooning by comparing to experiment 3
* Possibly show the swap values and the timing of usemem script.
* Ended at 11:42 on 29th

# Fifth
* To determine the correctness of migration on with physical machines
* 7 instances each on compute 3 and compute 2
* Started at 13:20 on 29th
