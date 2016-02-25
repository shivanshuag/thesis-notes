* Qemu allocates the guest RAM in a contiguous chunk of `maxmem` size using the `mmap` call.
* Using the `pmap` command, the RSS of that chunk can be found out. ex- If the `maxmem` of the guest is 4GB, run `sudo pmap -X [pid of qemu process] | less` and search for memory mapping of size `4194304` (4096*1024). The `RSS` of that mapping is the amount of physical RAM used by the guest VM.
* Test data -

maxmem - 4GB
currentmem - 2GB

  *  usedmem - 91MB
     RSS - 282052/1024 = 275MB
  * usedmem - 1046MB
    RSS - 1261276/1024 = 1231MB

  Difference is around 185
In above case difference between usedmem and RSS when i.e. overhead seems constant.

 * usedmem - 91MB
   RSS - 278896/1024 = 272MB
 * usedmem - 567
   RSS - 765076/1024 = 747
 * used - 1045
   RSS - 1224

Difference is around 180
 Again the overhead remains almost same :) .

maxmem - 2GB
currentmem - 1500MB

  * usedmem - 82MB
    RSS - 249596/1024 = 243MB
  * usedmem - 560MB
    RSS - 718MB

Difference is around 160.


So the overhead changes with change in maxmem. Is there a lower and upper bound on it?

maxmem - 8GB
currentmem - 8GB

  * usedmem - 97MB
    RSS - 345640/1024 = 337MB
  * usedmem - 574MB
    RSS - 830556/1024 = 811

  Difference is around 240MB

maxmem - 16GB
  * usedmem - 111MB
    RSS - 471100/1024 = 460MB

  * usedmem - 588MB
    RSS - 946944/1024 = 924MB

    Difference is 350MB
