# Virtio Balloon Code Walkthrough

In KVM, all the code for balloon driver is in the file `drivers/virtio/virtio_balloon.c`

Line 485 - `virtballoon_probe` function is run when the module is loaded.  
Line 496 - It initializes a `virtio_balloon` structure. Look at its definition in the same file.  
504 and 505 - Two new wait queues initialized. Learn about wait queues [here](https://lwn.net/Articles/577370/)  
514 - `init_vqs` function is called with `virtio_balloon` structure. This is defined in the same file and sets up the queue shared by the backend driver in the hypervisor
526 - Creates a new thread which starts at `balloon` function


384 - `balloon` function starts
359-357 - thread will wait on the config_change and will come out of the loop when the conditions in the `if` statement become true.
370-375 - calls `fill_balloon` or `leak_balloon` function based on the `diff`


In qemu, the code for balloon dirver is inside `hw/virtio/virtio-balloon.c`
