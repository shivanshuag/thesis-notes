# Virtual Machine Monitor

* Must have the control of all the hardware resources and should be able to take away resource from one VM and give it to other VMs.
* Must maintain the state of registers of all the VMs

## Example of Interval Timer

This timer is used by the OS for scheduling processes. Before passing control to a user process, OS sets this time to a the value after which OS would get a timer interrupt. This ensures that OS has the control of the hardware at all times.

Similarly, a VMM must emulate the Interval timer and set the hardware interval timer to the time after which VMM wants to take back the control. Thus, VMM must intercept access to all the privileged resources of the system and must not be directly usable by the guest VMs but should be emulated by the VMM.

## I/O virtualization


## Hosted VMM

* VMM-n (native): This component runs natively on the hardware and has characteristics similar to the VMM on a native virtual machine system. It is the component that intercepts traps due to privileged instructions or patched critical instructions encountered in a virtual machine. It may provide device drivers for a small set of devices that are either performance critical, or that do not have drivers already available in the host operating system.
* VMM-u (user): This component runs as a user-mode process on the host operating system. As mentioned earlier, this component makes resource requests to the host OS, in particular, memory and I/O requests, on behalf of the native mode VMM. VMM-u makes these requests using system library functions supplied with the host operating system.
* VMM-d (driver): This component provides a means for communication between the other two components. This is done by making the VMM-n appear to the VMM-u as a device attached to the host operating system. The VMM-d is essentially a special device driver installed on the host operating system, and it provides the link from VMM-u to VMM-n. The only user program on the host system allowed to access the VMM-n "device" is the VMM-u component.


## References

* Chapter 8 of Book by Smith, Nair - Virtual Machines
