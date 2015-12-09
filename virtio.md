# VirtIO

VirtIO provides an abstraction for paravirtualized drivers. This means that it provides a transport mechanism for communication of guest driver to host, mechanism for probing and configuring the devices. Using these, front end drivers(used in the guest OS) and corresponding backend drivers(present in the hypervisor) can be made.

## Structures in VirtIO

![Virtio abstractions](http://www.ibm.com/developerworks/library/l-virtio/figure4.gif)
Virtio abstraction consists of `virtio_driver` which has `probe` function to probe for devices for that driver. It has associated `virtio_devices`. `virtio_device` has `virtio_config_ops` which has operations for configuring the device. This has a `find_vq` function which gives `virtqueue` for the device. The find_vq function also permits the specification of a callback function for the virtqueue, which is used to notify the guest of response buffers from the hypervisor.

## Virtqueue

Virtqueue is used for communication to and from the backend driver. `virtqueue` has `virtqueue_ops` which has functions for all the operations that can be performed with the queue. `add_buf` adds a buffer to the queue. A buffer is a [scatter-gather array](https://en.wikipedia.org/wiki/Vectored_I/O) which is an array of buffers containing data all of which are added to the queue in order. `add_buf` also takes an argument called `data` which is returned by the `get_buf` call when this buffer has been consumed. `kick` function performs a VMExit and notifies the backend driver that there are buffers to consume. `get_buf` is used to get which buffer has been consumed by the host. It return the `data` that was passed along with the buffer in the `add_buf` call.

`disable_cb` disables the callbacks(which was provided in the find_vq call) of buffers consumed by the hypervisor. `enable_cb` does the  opposite. `enable_cb` returns false if a buffer was consumed by the hypervisor/host after the last `get_buf` call i.e. there is a consumed buffer in the virtqueue. The typical workflow is to disable callbacks, get all the consumed buffers and enable callbacks again so that any new consumed buffer will transfer the control to callback function. But if a buffer is added after the `get_buf` call and before `enable_cb`, there is a possibility that it will never be acknowledged.

## References

* http://www.ibm.com/developerworks/library/l-virtio/
* http://dl.acm.org/citation.cfm?id=1400097.1400108
