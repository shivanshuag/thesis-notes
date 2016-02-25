# Testing DRS

## Testing the Load filtering Algo

* For each host, plot a graph of load profile along with the points where data is updated in etcd.
* Compare this with graph where etcd is updated without filtering i.e. threshold based.
* Generate load profile similar to how VMware does it.

## Testing load on etcd

* Check out https://github.com/coreos/etcd/blob/master/Documentation/metrics.md and create a plan for testing etcd.

## Testing Auto Ballooning

* Testing without migrations turned on.
* Compare situations without auto-ballooning disabled and with auto-ballooning enabled.
* Count number of time migration is needed.

## Testing with migration turned on

* Metrics on time taken for migration
* Number of migrations plotted with time
*
