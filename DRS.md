# Ideas

* First step is to identify memory overload in any guest. The way to do this will be to identify low memory on the host.
* First try ballooning. If cannot reclaim from any guest, then consider migrating.
* Algorithm should be stable, i.e. a machine should not keep on migrating, so while selecting a VM, take into account the number of migrations it has had before.
* If a machine is migrated, it should have some room to grow.
* Think some way for CPU. Like if CPU is overcommitted, you can migrate one VM so that one whole core is available to the other VM.

* Assign weights to all the factors mentioned above, and then calculate an overall score which represents whether a machine should be migrated/ which machine should be migrated.

* VMware DRS checks every 5 minutes. 5 minutes can be a long time. Have 2 kinds of balancer - immediate and long term. Immediate scheduler can be decentralized (p2p) and can take decisions on its own to reduce contention. Long term balancer can be centralized, which tries to reduce the overall load imbalance in the cluster.
  * Is 5 minutes really that large a time interval? Migration has a cost associated with it, so should not migrate on spikes of short time duration. Should migrate only when resource contention happens for some time. What should be ideal time interval for deciding migration?
  * How to coordinate short term and long term scheduler? Short term scheduling should not happen when long term scheduler is running.
  * Short term scheduler can be proactive, while long term will be reactive i.e. triggered when a suitable migration candidate is not found by any of the PM's.
  * How to calculate CPU demand?

* Self balancing cluster using decentralized load balancing. Each PM asks for info from all other machines, decides where to send and then sends it.
    * What if no suitable candidates found for migration?
    * What if multiple migrations are needed to resolve the imbalance? I think this problem can be solved. Each individual migration will improve the imbalance of the cluster in some way, which is what happens in VMWare DRS. The difference is that VMWare DRS chooses the best migration(migration which reduces the imbalance most) to take place, while this will allow every migration where benefit > cost.
    * How to ensure that the decentralized algorithm is stable i.e there is no chain of migration? Increase the cost of migration as the count of migration for a particular VM increases and give the VM room to grow. Decide cost according to the priority/share of the VMs.
    * What about initial placement of VM? Same problem as VM migration.
    * How to calculate resource entitlement and hence discover resource imbalance? How to ensure that a VM does not starve and there is fair usage of resources based on priority/shares assigned to each VM? No need to calculate demand. Each Virtual Machine has a size and allocation. When it starts using all the resources that are allocated to it, and its allocation < size, then possibly  migrate. Fetch data from the data collection service. Calculate resource entitlement using shares and usage of each VM as its demand. The demand of the VM to be migrated should be 10%(configurable) more than its usage. If the allocation is more than entitlement, do nothing(some other VM will be migrated here to fill the empty room here, meanwhile the empty room is being used by the VMs on this PM). Else, migrate it to the host which gives most room to grow.
    ~~Calculation of entitlement - Need to know the demand of each VM for this. How to do this in a de-centralized manner?  Lets have priorities for each VM say 1-10. Ensure that the mean priority of VMs on a PM is as close to the mean priority of the cluster a possible. No need for shares, priority decides how much resource is allocated. When there is a unsatisfiable demand, query for data of other servers. Lets say that the data resides on a central server which collects usage data from all PM's and any PM can query it to get the usage data of all other VMs.In that case, no need for priorities. We can have shares.~~
    * How to tackle unstable workloads? VMWare predicts the demand change of workload based on previous 1 hour history.
    * ~~Local migrations would incur less cost, so incorporate locality in this. Something like - first it queries machines nearby, then machines farther off.~~
    * What should be a metric to measure performance of DRS? Maybe weighted mean(where weights are priority) of the product of the demands met and time for each VM. But this is also workload dependent. So the demands satisfied for specific type of workload tells how the DRS in performs in which situation. Also need to incorporate number of migrations, the cost the migrations incur.

    Advantages/disadvantages of decentralized resource management -

    Lets take two scenarios.
    1. The cluster is underloaded i.e. the sum of demand of all the VMs is less than cluster capacity. So, the demands of all the VMs can be fulfilled. Chances are that when a VM needs to be migrated, a suitable candidate can be found. Give examples of cases where you can't find a suitable candidate even if cluster is underloaded.
    2. The cluster starts to get overloaded. So, this will start with imbalance on some machine. It will fetch info form the central server, find out the imbalance and will decide then whether to migrate or not. In the previous case, it was sure to migrate, but here it might turn out that the resource the VM is entitled to is satisfied here itself. The whole algorithm is like running the the central DRS on the machines, does it make sense? What advantages can we get out of it? Here, we try to do something only when there is a need, no unnecessary migrations. VMWare migration tries to reduce imbalance, even when the demands of VMs are satisfied, they will be migrated. Here, no such migrations. But this similar thing can be done in centralized model.


# Design of DRS

1. A data collection service for openstack. Each of the compute nodes sends its stats when it joins the cluster. The stats are - Number of VMs, CPU usage each VM, Memory usage of each VM, number of times a VM has been migrated in the past hour, shares/priority of each VM. After that, only when the demands change by some threshold is the change reported back to the datastore. Take help from cielometer and develop your own service. Decide on a database backend to be used. Cassandra seems good for this purpose. Or simple postgres.
2. A memory overcommitment manager service which balloons the VMs according to their shares on a PM. So Memory Overcommitment and DRS work orthogonally. DRS tries to migrates VMs if their requirements are not satisfied. Overcommitment Manager balloons the VMs on a PM. No need of any outside information for this. Just use the shares of VM on a particular PM.
3. A DRS service which detects if there are unsatisfied demands. Then fetches data from the data collection service, finds if there is need for migration, then migrates to the best candidate.

Problems with this algo - No consolidation

## The Monitoring Service

* There are four types of memory for a VM.
  * maxmem - The maximum memory that can be allocated to a VM
  * currentmem - Memory of the VM after ballooning
  * usedmem - The amount of memory which is being used by the VM
  * actualmem - The memory of the VM backed by physical RAM
  Since on freeing memory, the VM has no way of letting the hypervisor know that it has free some memory, actualmem >= usedmem. Ballooning takes pages from the (actualmem - usedmem) first and hands them back. After that ballooning need not do anything, because it can just decrease the currentmem without having to free any pages. When the currentmem is set to < usedmem, then some more physical pages are freed and their content is swapped by the VM.
* The service which handles ballooning first takes the `actualmem-usedmem` from each of the VMs. This should work in case of no overload. If there is overload, then calculate entitlement.

# Progress

* ~~read about irqfd and ioeventfd~~ http://blog.allenx.org/2015/07/05/kvm-irqfd-and-ioeventfd/
* ~~read about memory compaction~~  https://lwn.net/Articles/368869/
* read qemu code
* understand qemu memory representation etc. (in progress)
* ~~Read paper and other resources on virtio~~
* ~~Xen selfballooning, transcendent memory~~
* ~~VMWARE BALLOON~~ (turned out to be useless, as the code is closed source, only ABI's are exposed)
* ~~Check out Memory Overcommittment Manager~~
* ~~Install xen and try out ballooning~~
* ~~virtio balloon vs xen balloon~~
* ~~ESX memory overcommitment~~ https://labs.vmware.com/vmtj/memory-overcommitment-in-the-esx-server
* ~~Read VMWare DRS http://www.waldspurger.org/carl/papers/drs-vmtj-mar12.pdf~~
* ~~See how to get balloon driver stats~~ read guest-monitoring for commands on how to get stats.
* ~~See how to use libvirt events according to https://github.com/wiedi/libvirt/blob/master/examples/domain-events/events-python/event-test.py and https://github.com/libvirt/libvirt/blob/master/include/libvirt/libvirt-domain.h~~
* How much buffer cache to keep? Currently 50% is kept
* ~~Check out https://raw.githubusercontent.com/pixelb/ps_mem/master/ps_mem.py~~ Gets the memory used by each process, but considers all processes of one type as one. So, no way to get per VM actualmem from it. Can be modified maybe?
* ~~Get per vm actualmem~~
* Find out a way to get qemu overhead for a VM. Will be useful in calculating idle memory.
* Find a way to use available memory - https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=34e431b0ae398fc54ea69ff85ec700722c9da773


* Read E-PVM, a method for assigning scores on where to place a task (http://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=877834) (http://gec.di.uminho.pt/discip/minf/ac0203/icca03/costappr_tpds2000.pdf) - convert the total usage of several heterogeneous resources,
such as memory and CPU, into a single homogeneous “cost.” Jobs are then assigned to the machine where they have the lowest cost.
* Read netwrok IO control - http://www.vmware.com/files/pdf/techpaper/VMW_Netioc_BestPractices.pdf
* Read https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Gulati.pdf

# Resources

* 2012 presentation on developments in memory management in KVM - http://www.linux-kvm.org/images/1/19/2012-forum-memory-mgmt.pdf

# Papers on resource distribution
* http://www.hindawi.com/journals/mpe/2013/878542/
* http://www.serc.iisc.ernet.in/~jlakshmi/Research/CloudsandQoS/VMPlacementOptimizationCloudcom2013.pdf
* http://henricasanova.github.io/papers/stillwell_ipdps12.pdf
* https://hal.archives-ouvertes.fr/hal-00474721/document
* http://weblab.ing.unimo.it/papers/cloudcomp09.pdf
* cloudscale - http://www.e-wilkes.com/john/papers/2011-CloudScale-SoCC.pdf
* sandpiper - https://people.cs.umass.edu/~arun/papers/sandpiper.pdf
* EPVM - http://gec.di.uminho.pt/discip/minf/ac0203/icca03/costappr_tpds2000.pdf
