# Ideas

* First step is to identify memory overload in any guest. The way to do this will be to identify low memory on the host.
* First try ballooning. If cannot reclaim from any guest, then consider migrating.
* Algorithm should be stable, i.e. a machine should not keep on migrating, so while selecting a VM, take into account the number of migrations it has had before.
* If a machine is migrated, it should have some room to grow.
* Think some way for CPU. Like if CPU is overcommitted, you can migrate one VM so that one whole core is available to the other VM.

* Assign weights to all the factors mentioned above, and then calculate an overall score which represents whether a machine should be migrated.


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
* Read VMWare DRS http://www.waldspurger.org/carl/papers/drs-vmtj-mar12.pdf
* See how to get balloon driver stats

# Resources

* 2012 presentation on developments in memory management in KVM - http://www.linux-kvm.org/images/1/19/2012-forum-memory-mgmt.pdf
