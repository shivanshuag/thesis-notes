
Following session shows how to retrieve guest memory statistics from a qemu guest.


```
virsh # connect qemu:///system

virsh # qemu-monitor-command debian8 --pretty '{"execute":"query-kvm"}'
{
    "return": {
        "enabled": true,
        "present": true
    },
    "id": "libvirt-99"
}


virsh # qemu-monitor-command debian8 --pretty {"execute":"qmp_capabilities"}
error: internal error: cannot parse json {execute:qmp_capabilities}: lexical error: invalid char in json text.
                                      {execute:qmp_capabilities}
                     (right here) ------^


virsh # qemu-monitor-command debian8 --pretty '{"execute":"qmp_capabilities"}'\
error: dangling \
virsh # qemu-monitor-command debian8 --pretty '{"execute":"qmp_capabilities"}'
{
    "id": "libvirt-100",
    "error": {
        "class": "CommandNotFound",
        "desc": "Capabilities negotiation is already complete, command 'qmp_capabilities' ignored"
    }
}


virsh # qemu-monitor-command debian8 --pretty '{"execute":"qom-list"}'
{
    "id": "libvirt-101",
    "error": {
        "class": "GenericError",
        "desc": "Parameter 'path' is missing"
    }
}


virsh # qemu-monitor-command debian8 --pretty '{"execute":"qom-list",  "arguments": { "path": "/machine/peripheral/balloon0" }}'
{
    "return": [
        {
            "name": "virtio-pci[1]",
            "type": "child<qemu:memory-region>"
        },
        {
            "name": "virtio-bus",
            "type": "child<virtio-pci-bus>"
        },
        {
            "name": "virtio-pci-cfg[0]",
            "type": "child<qemu:memory-region>"
        },
        {
            "name": "virtio-pci[0]",
            "type": "child<qemu:memory-region>"
        },
        {
            "name": "bus master[0]",
            "type": "child<qemu:memory-region>"
        },
        {
            "name": "guest-stats-polling-interval",
            "type": "int"
        },
        {
            "name": "guest-stats",
            "type": "guest statistics"
        },
        {
            "name": "any_layout",
            "type": "bool"
        },
        {
            "name": "notify_on_empty",
            "type": "bool"
        },
        {
            "name": "event_idx",
            "type": "bool"
        },
        {
            "name": "indirect_desc",
            "type": "bool"
        },
        {
            "name": "deflate-on-oom",
            "type": "bool"
        },
        {
            "name": "virtio-backend",
            "type": "child<virtio-balloon-device>"
        },
        {
            "name": "parent_bus",
            "type": "link<bus>"
        },
        {
            "name": "command_serr_enable",
            "type": "bool"
        },
        {
            "name": "multifunction",
            "type": "bool"
        },
        {
            "name": "rombar",
            "type": "uint32"
        },
        {
            "name": "romfile",
            "type": "str"
        },
        {
            "name": "addr",
            "type": "int32"
        },
        {
            "name": "legacy-addr",
            "type": "str"
        },
        {
            "name": "disable-modern",
            "type": "bool"
        },
        {
            "name": "disable-legacy",
            "type": "bool"
        },
        {
            "name": "virtio-pci-bus-master-bug-migration",
            "type": "bool"
        },
        {
            "name": "class",
            "type": "uint32"
        },
        {
            "name": "hotplugged",
            "type": "bool"
        },
        {
            "name": "hotpluggable",
            "type": "bool"
        },
        {
            "name": "realized",
            "type": "bool"
        },
        {
            "name": "type",
            "type": "string"
        }
    ],
    "id": "libvirt-102"
}


virsh # qemu-monitor-command debian8 --pretty '{"execute":"qom-set",  "arguments": { "path": "/machine/peripheral/balloon0","property":"guest-stats-polling-interval","value":2}}'
{
    "return": {

    },
    "id": "libvirt-103"
}


virsh # qemu-monitor-command debian8 --pretty '{"execute":"qom-get",  "arguments": { "path": "/machine/peripheral/balloon0","property":"guest-stats-polling-interval"}}'
{
    "return": 2,
    "id": "libvirt-104"
}


virsh # qemu-monitor-command debian8 --pretty '{"execute":"qom-get",  "arguments": { "path": "/machine/peripheral/balloon0","property":"guest-stats"}}'{
    "return": {
        "stats": {
            "stat-swap-out": 0,
            "stat-free-memory": 4062081024,
            "stat-minor-faults": 53634,
            "stat-major-faults": 260,
            "stat-total-memory": 4158361600,
            "stat-swap-in": 0
        },
        "last-update": 1453032472
    },
    "id": "libvirt-105"
}


virsh # qemu-monitor-command debian8 --pretty '{"execute":"qom-get",  "arguments": { "path": "/machine/peripheral/balloon0","property":"guest-stats"}}'
{
    "return": {
        "stats": {
            "stat-swap-out": 0,
            "stat-free-memory": 4061319168,
            "stat-minor-faults": 54205,
            "stat-major-faults": 263,
            "stat-total-memory": 4158361600,
            "stat-swap-in": 0
        },
        "last-update": 1453032530
    },
    "id": "libvirt-106"
}
```

This does not give the memory in buffer cache. That option has to be added.
