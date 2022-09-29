## Config WAN-LAN L3 layer SFE offload

### Topology

	PC1(192.168.2.100)--(192.168.2.1 WAN eth0)Ap Router(br-lan 192.168.1.1 LAN eth1)--PC2(192.168.1.175)
	LAN under br-lan,PC2 will get the ip(192.168.1.175)  
	Config WAN interface as below:

		config interface 'lan'
	        option ifname 'eth1'
	        option force_link '1'
	        option type 'bridge'
	        option proto 'static'
	        option ipaddr '192.168.1.1'
	        option netmask '255.255.255.0'
	        option ip6assign '60'
	        option multicast_querier '0'
	        option igmp_snooping '0'
	        option ieee1905managed '1'

		config interface 'wan'
	        option ifname 'eth0'
	        option proto 'static'
	        option ipaddr '192.168.2.1'
	        option netmask '255.255.255.0'

	Config PC1 static IP: 192.168.2.100 
	Config PC1 PC2 router table:
		PC2 : route ADD 192.168.2.100 MASK 255.255.255.255 192.168.2.1 IF 5
		PC1 : route ADD 192.168.1.175 MASK 255.255.255.255 192.168.1.1 IF 12
		//interface id you can get from 'router print', you can also use 'router print' to check whether router add successfully. 
		//Pay attention: you need run cmd as administrator

	Config AP
		route add -host 192.168.2.100 gw 192.168.2.1  dev eth0 //iperf -c 192.168.2.100
		//Exit BIG-IP Edge 
		//If ping failed, you can del the route rule and add again
		route del -host 192.168.2.100 gw 192.168.2.1  dev eth0

		root@OpenWrt:/# route
		Kernel IP routing table
		Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
		192.168.1.0     *               255.255.255.0   U     0      0        0 br-lan
		192.168.2.0     *               255.255.255.0   U     0      0        0 eth0
		192.168.2.100   192.168.2.1     255.255.255.255 UGH   0      0        0 eth0


### Enable SFE 
	vim /etc/config/ecm 
	config ecm 'global'
        option acceleration_engine 'sfe'
	
### Test log
	you can use iperf generate the traffic 
	 <stats num_connections="0" pkts_forwarded="835348" pkts_not_forwarded="3959" create_requests="1377" create_collisions="0" 
	 destroy_requests="1375" destroy_misses="0" flushes="1377" hash_hits="835350" hash_reorders="0" />
	Pay attention: SPF11.5 don't support L2 offload

### Debug way
	1. sfe_dump get the accelerated rules 
	2. SFE driver also use dynamic to get the log just like ECM 
		echo 8 > /proc/sys/kernel/printk
		echo "file ecm_ported_ipv4.c +p" >/sys/kernel/debug/dynamic_debug/control
		echo "file ecm_sfe_ported_ipv4.c +p" >/sys/kernel/debug/dynamic_debug/control
		echo "file ecm_sfe_ipv4.c +p" >/sys/kernel/debug/dynamic_debug/control

		echo "file sfe_ipv4.c +p" >/sys/kernel/debug/dynamic_debug/control
		echo "file sfe_cm.c +p" >/sys/kernel/debug/dynamic_debug/control
	3. SFE statistics
		cat /sys/kernel/debug/ecm/ecm_sfe_ipv4/*
		cat /sys/kernel/debug/ecm/ecm_sfe_ipv6/*
