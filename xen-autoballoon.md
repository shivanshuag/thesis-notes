# Auto-ballooning in XEN

## Xen Transcendent Memory

Allows RAM to be shared across kernels. "The end goal is that memory can be more efficiently utilized by one kernel and/or load-balanced between multiple kernels". Implementation is divided into into front-end and back-end. Front-end implementations are `frontswap` for anonymous pages and `cleancache` for file backed pages.

The two basic operations of tmem are "put" and "get". If the kernel wishes to save a chunk of data in tmem, it uses the "put" operation, providing a pool id, a handle, and the location of the data; if the put returns success, tmem has copied the data. If the kernel wishes to retrieve data, it uses the "get" operation and provides the pool id, the handle, and a location for tmem to place the data.

### Frontends

Frontswap allows the Linux swap subsystem to use transcendent memory, when available, in place of sending data to and from a swap device.  If tmem rejects it, the swap subsystem writes the page, as normal, to the swap device.

Mapped pages can be reclaimed by the kernel any time. If the same page is to be used again, page fault happens and it is fetched from the disk. Using tmem, the page to be reclaimed can be put in the `cleancache` and can be reclaimed from there if it used again. Cleancache data in tmem is ephemeral, which means that the page of data may be discarded if tmem chooses. Later, if the kernel determines it needs that page of data after all, it asks tmem to give it back. If tmem has retained the page, it gives it back; if tmem hasn't retained the page, the kernel proceeds with the refault, fetching the data from the disk as usual.

There are ways to flush frontswap cache and cleancache i.e. remove/invalidate a page sotred there.

### Backends

Backends for tmem include `zcache` and `Xen tmem`. `Zcache` can be used in a non-virtualized environemnt. When the kernel has N pages to store but available memory to store them is less than N*pagesize, it can put them in tmem. Zcache compresses the pages before storing, so more pages can be stores. it usually rejects pages which have a low compression factor to avoid storing poorly compressible data.

 The tmem backend in Xen utilizes spare hypervisor memory to store data, supports a large number of guests, and optionally implements both compression and deduplication (both within a guest and across guests) to maximize the volume of data that can be stored.

 When kernel swaps a page, it assumes that the page will go to disk and may remain there for long time even if it is not used again as kernel assumes disk space is less costly and abundant. But if the page has gone to frontswap, it is taking up valuable space. To resolve this problem, `frontswap-self-shrinking` is used. When a guest is under normal memory pressure, this reclaims pages from tmem and brings it back to the kernel's RAM (HOW??).

### Conclusion

 Tmem does not seem to be useful during memory crunch on the host. But when memory is available on the host, then it may speed up by handling swapping if the balloon driver is slow to kicks-in and increase memory available or hard limit for ballooning has been reached on the guest. If there is no space available, then migration seems to be better option.

## Auto-ballooning

Read its code in `drivers/xen/selfballoon.c`

Auto-ballooning requires transcendent memory to be enabled in XEN. Simple implementation. A process runs regularly after a fixed time interval(configurable) which sets the target size. Target Size = Committed pages + reserved pages + balloon reserved pages (used to reserve pages for cache etc. with default value = 10% of the total ram). If the target is less than current ram, guest is ballooned down, else it is ballooned up. There is a hysteresis counter which represents the number of iterations it will take for machine to balloon down to target. So, each time self-ballooning process runs, Ram = current - (current-target)/hysteresis_counter. Hysteresis counter is different for ballooning up and down.

There is a `min usable mb` parameter which specifies the minimum amount of ram the guest should have. Machine cannot be ballooned down below this point.

This kind of ballooning is useful in XEN because all the idle memory can be used as transcendent memory, while in KVM, there is no tmem, so use in giving memory back to host if there is no use for it.

Self-ballooning also does frontswap shrinking along with ballooning.

## References
* https://lwn.net/Articles/454795/
