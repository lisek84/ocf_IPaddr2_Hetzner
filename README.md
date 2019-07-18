# ocf_IPaddr2_Hetzner
Pacemaker/Corosync OCF Resource Agent for Hetzner DC or any other with routing tables required

This is Modified version of offical IPaddr2 script.

It allows you to use additional routing tables for failover IP's

It's named Hetzner cause it's prepared to work for them. BUT it can be used anywhere where you need to add routing tables!

It's prepared to match this Hetzner Manual:
https://wiki.hetzner.de/index.php/Vswitch/en


# How to install it

(You can replace custom with anything you like, but remember about it when configuring provider name in pacemaker)
```
mkdir /usr/lib/ocf/resource.d/custom/
```
There copy file IPaddr2_Hetzner
```
chmod 755 /usr/lib/ocf/resource.d/custom/IPaddr2_Hetzner
```
Also execute this on all of your machines in cluster
```
echo "1 vswitch" >> /etc/iproute2/rt_tables
```


# Configure Pacemaker

This is just as IPaddr with one additional required parameter - a routing_table (default is vswitch if you miss it)
Also remember about provider name as provider=(custom in this case) or ocf:custom:IPaddr2_Hetzner


example configuration:
```
primitive failover1 ocf:custom:IPaddr2_Hetzner \
        params ip=111.222.333.444 nic=eno1.4022 cidr_netmask=27 routing_table=vswitch \
        op monitor on-fail=restart interval=10s \
        op start interval=0 timeout=20s on-fail=restart \
        meta failure-timeout=30s migration-threshold=5 target-role=Started
```

Also After configuring this. You need to use default pacemaker's ocf Route Agent to handle default route for your table. This one supports routing tables by default so no need to do anything about this.

```
primitive gw Route \
        params destination=default gateway=111.222.333.111 table=vswitch family=ipv4 \
        op monitor on-fail=restart interval=10s \
        meta failure-timeout=30s migration-threshold=5 is-managed=true target-role=Started
```

## Authors
* **ClusterLabs** (https://github.com/ClusterLabs)
* **Lisek** (https://github.com/lisek84)

## License

This project is licensed under the GNU License
