# Using PF_RING packages
At http://packages.ntop.org we build binary PF_RING packages ready to use.

An option for configuring PF_RING and drivers is doing it through the nBox GUI. 
In case you are not using the nBox GUI, you can manually configure the packages 
from command line, as described below.

Once the "pfring" package, and optionally the "pfring-drivers-zc-dkms" package
containing ZC  drivers, is installed following the procedure on the web page, 
it is possible to use the init script /etc/init.d/pf_ring to automate the kernel 
module and drivers loading. The init script acts as follows:

1. loads the pf_ring.ko kernel module.
2. scans the folders /etc/pf_ring/zc/{e1000e,igb,ixgbe,i40e,fm10k}/ searching files:
- {e1000e,igb,ixgbe,i40e,fm10k}.conf containing the driver parameters
- {e1000e,igb,ixgbe,i40e,fm10k}.start that should be just an empty file
3. loads the drivers whose corresponding {e1000e,igb,ixgbe,i40e,fm10k}.start file is present, 
   unloading the vanilla driver.
2. configures hugepages if a ZC driver has been loaded, reading the configuration 
   from /etc/pf_ring/hugepages.conf. Each line (one per CPU) of the configuration file 
   should contain:
```
node=<NUMA node id> hugepagenumber=<number of pages>
```

Note: in order to figure out what is the driver model that you need, please use
ethtool -i <interface>. Example:

```
ethtool -i eth1 | grep driver
driver: ixgbe
```

Below you can find a configuration example for a dual-port ixgbe card (replace
ixgbe with your actual driver model) with ZC drivers, for other packages/drivers 
configuration steps are similar.

```
mkdir -p /etc/pf_ring/zc/ixgbe
echo "RSS=1,1" > /etc/pf_ring/zc/ixgbe/ixgbe.conf 
touch /etc/pf_ring/zc/ixgbe/ixgbe.start
touch /etc/pf_ring/pf_ring.conf
touch /etc/pf_ring/pf_ring.start
echo "node=0 hugepagenumber=1024" > /etc/pf_ring/hugepages.conf 
tree /etc/pf_ring/
|-- hugepages.conf
|-- pf_ring.conf
|-- pf_ring.start
`-- zc
    `-- ixgbe
        |-- ixgbe.conf
        `-- ixgbe.start
```

Please note that in this configuration RSS is disabled (RSS=1 means single queue). 
For learning more about RSS and enable multiple queues for hw traffic distribution 
please read the RSS guide.

In order to run the init script, after all the files have been configured:

```
/etc/init.d/pf_ring start
```

You can check that the ZC driver is actually running with:

```
cat /proc/net/pf_ring/dev/eth1/info | grep ZC
Polling Mode:      ZC/NAPI
```

