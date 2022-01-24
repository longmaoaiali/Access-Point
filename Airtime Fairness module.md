# Airtime Fairness (ATF) 
<p align="right"> 1/24/2022 9:30:11 AM </p>
	ATF focuses on scheduling fairness for transmission of traffic from Access Point (AP), and efficient Wi-Fi bandwidth utilization.
	Its algorithm does not deal with transmission of frames from other clients.

## 1.ATF Principle

### 1.1.ATF Change TX Scheduler
	Each ‘Token’ is a unit of time and each token equates to 1 microsecond.
#### 1.1.1.Tx duration estimation: Estimate Tx duration for the packet before transmit
	The Tx duration estimate happens before each packet/aggregate transmit. 
	Before adding the packet to the hardware queue, the packet duration is estimated by the routine ath_pkt_duration. 
	Best MCS rate is considered for the duration estimate. The routine returns the duration in micro seconds. 
	Transmission overheads including, SIFS, backoff and ack duration is added to the value returned by the routine. 

	The estimated duration is now compared with the tokens available for this node. 
	If the estimated duration is less than the token available, the node token is updated (after deducing the estimated duration) and the buffer is send to the hardware for transmit. 
	The estimated duration is also updated in the Tx descriptor. 
	On the other hand, if the estimated duration is more than the tokens available, the packet is enqueued to the software queue and will be send out on the next token availability.

#### 1.1.2.Tx duration actual: Calculate actual packet duration on transmit completion
	On receiving a transmit completion indication, the actual Tx duration for the packet is calculated. 
	The MCS rate at which the packet was actually transmitted & the number of retries are read from the Tx descriptor. 
	The actual Tx duration is then compared with the estimated duration already available in the descriptor. 

	If the actual duration is more than the estimated duration (in case of retries or transmit at a lower MCS rate), the difference is deducted from the node tokens in the next token replenish cycle. 
	If the estimate duration is more than the actual duration, the difference is added to the node tokens in the next token replenish cycle.

### 1.2.airtime fairness influence TX Scheduling(Tx Tokens) 
	Each ‘Token’ is a unit of time and each token equates to 1 microsecond.
	1. After the ATF table is updated following a user configuration or client join/leave, update_atf_nodetable routine iterates through each peer entries in the ATF table and finds the corresponding entry in the node table (struct ieee80211_node).
	2. The routine then reads the ATF unit allotted to the peer (ic->atfcfg_set.peer_id[i].sta_cal_value) and updates this in the node structure.
	3. A timer, atf_tokenalloc_timer configured to run every 200 milliseconds, iterates through the node table. This timer routine converts atf units to tokens that can be passed on to the scheduler. 
	4. For each valid node entry in the node table, the timer reads the atf unit and calls the routine ieee80211_distribute_atf_txtokens to distribute tokens. The tokens are passed on to the LMAC layer. 
	
	Tx tokens = atf_unit * (Timer Interval in micro seconds) / ATF DENOMINATION
	where ATF DENOMINATION is set to 1000. The Choice to use 1000 ( 1 part of 1000) rather than percentage ( 1 part of 100) will help us to avoid dealing with floating point arithmetic and thus less approvimation.
	Thus, for a peer with 100% atf allotted, txtokens are 200000 (1000 * (200 * 1000) /1000 = 200000 usec)

### 1.3.Free token collection and distribution algorithm
#### 1.3.1.Collection Phase: 
	High level design change at free token collection level is as follows:
		1.Collection algorithm operates only on peers which are marked as fair and collects the unused tokens from last ATF interval.
		2.Collected tokens are accumulated at PDEV level. Since free tokens are collected from fair peers a global pool can be maintained and distributed from a singular location.
	
#### 1.3.2.Distribution Phase:
	High level design change at free token distribution level is as follows:
		1.When a peer exhausts its tokens, free tokens are allocated to them from the cumulative free tokens pool from the group where it belongs. 
		If the parent group does not contain enough free tokens they are borrowed from the neighboring group if it contains free tokens.
		2.Each node in the hierarchy has a used token counter which accounts the tokens used every ATF interval. 
		During distribution, a peer can be denied free tokens if the used token value exceeds the restricted airtime configured in Restricted fair mode.


## 2.ATF Configurations Command

### 2.1.Enable/Disable ATF
	1. cfg80211tool athX commitatf 1/0   //effect the input setting; this wakes up the ATF time schedule task.
    2. cfg80211tool athX get_commitatf   //show whether enable ATF 

### 2.2.ATF Scheduling 
	cfg80211tool wifi0 atfstrictsched 0 //fair scheduling
	cfg80211tool wifi0 atfstrictsched 1 //strict scheduling
	cfg80211tool wifi0 gatfstrictsched //verify strict or fair scheduling of the Airtime Fairness. 1 Strict scheduling or 0 Fair scheduling
	
	cfg80211tool athN atfssidsched 1/0 //enable or disable ATF strict scheduling per SSID

### 2.3.List STAs who associated AP
	wlanconfig ath0 list sta 

### 2.4.ATF Configuration based SSID    
	The correct command sequence to assign airtime to a VAP and STA is as follows:
    1. Assign airtime to VAP
        wlanconfig ath0 addssid vap1 100 // Assigns 100% airtime to VAP1
    2. Assign airtime to STA
        wlanconfig ath0 addsta 000102030405 10 vap1 //Assign 10% airtime to specified MAC 00:01:02:03:04:05 who associated to VAP
    3. wlanconfig ath0 showairtime //show ATF percentage
    4. wlanconfig ath0 showatftable //show airtime configurations table
		SSID          Client(MAC Address)    Air time(Percentage)        
		ATF-SSID-0                                 100.0              
					  00:03:7f:12:9a:9a            80.0               
					  00:03:7f:12:6a:77            20.0     

### 2.5.ATF Debug
	1. cfg80211tool wifi0 atf_log 1/0 // enable/disable ATF logging
	
	2. dump the token allocation per subgroup
		wifitool <vap> atf_debug_size 128 // Set debug log size to 128
		wifitool <vap> atf_dump_debug // Dump debug logs for all subgroup.
		wifitool <vap > atf_debug_nodestate <node mac> // Display the node pointer to subgroup mapping and also the TID state. 
	
	3. Enable collection of ATF statistics
		PPDU duration per PDEV will be retrieved from htt_ppdu_stats_common_tlv->phy_ppdu_tx_time_us as it is without resp_time.
		Airtime per peer level in SU-MIMO is same as PPDU duration.
	
		PPDU duration per peer or user using the following formula in MU-MIMO PPDU duration per user = ppdu_duration * MU_ratio * OFDMA_ratio
		MU_ratio = usr_nss / Sum_of_nss_of_all_users
		OFDMA_ratio = usr_ru_width / Sum_of_ru_width_of_all_users
		usr_ru_widt = ru_end – ru_start + 1
		
		API: 
			atf_stats_print - print function 
			ucfg_atf_enable_stats() - API to enable ATF stats collection
			ucfg_atf_is_stats_enabled() - API to get ATF stats enabled or disabled
			tgt_atf_process_ppdu_stats() - API to process PPDU stats
		
		wlanconfig athX showatfstats //Shows ATF stats per SSID or Group along with list of associated Peers
		wlanconfig athX showatfacstats //Shows ATF stats based on AC configurations with respect to each group
		cfg80211tool athX enable_atf_stats 1 //Enable ATF stats feature
		cfg80211tool athX g_enable_atf_stats //Get the status of ATF stats feature.
		cfg80211tool athX atf_stats_timeout <10 - 50> //Set ATF stats timeout period. The default value is 30 seconds. 
		cfg80211tool athX g_atf_stats_timeout //Get ATF stats timeout period.
		

### 2.6.AC-based ATC commands
	Command to configure AC-based ATF Configuration
	1. wlanconfig <VAP> atfaddac <ssid/group> <ac>:<airtime> <ac>:<airtime>
	Example. wlanconfig ath0 atfaddac ssid1 BE:30 BK:20 VI:10 VO:15
	2. wlanconfig <VAP> atfdelAC <ssid/group> <ac>:<airtime> <ac>:<airtime>
	Example. wlanconfig ath0 atfdelAC ssid1 BE:30 BK:20 VI:10 VO:15

### 2.7.Command to configure per group scheduling policy
	wlanconfig <VAP> atfgroupsched <group name> <sched policy> sched policy : 0/1/2 (0 – Fair 1- Strict 2- FAIR UB or restricted-fair)
	Example. wlanconfig ath0 atfgroupsched group1 2 // configure ‘FAIR with UB’ or restricted-fair mode scheduling policy for group1

	wlanconfig ath0 showatfgroup // display group sched policy 

### 2.8.Command to configure per SSID scheduling policy
	cfg80211tool <vap> atfssidsched <sched policy> sched policy : 0/1/2 (0 – Fair 1- Strict 2- FAIR UB or restricted-fair mode)
	Example: cfg80211tool ath11 atfssidsched 2 //configures ath11 with ‘FAIR_UB’ or restricted-fair mode scheduling policy

### 2.9.Command to display groups/subgroups created
	wlanconfig <VAP> showatfsubgroup
	Example: wlanconfig ath0 showatfsubgroup

### 2.10.Group config commands
	1. Before adding any group, enable “atfssidgroup” using this command.
		cfg80211tool athX atfssidgroup 1
	2. Add new SSID to group with group_name. Create group if group not exists.
		wlanconfig athX addatfgroup <group_name> <SSID>
	3. Configure Group Scheduling policy (Optional)
		wlanconfig athX atfgroupsched <group_name> <sched_policy>
	4. Configure Groups with airtime
		wlanconfig athX configatfgroup <group_name> <airtime>
	5. (Optional) Configure groups with airtime% per WMM AC in current group. Given airtime% is a percentage of (Group airtime% - All explicit peers airtime%)
		wlanconfig athX atfaddac <group_name> VI:<airtime> BE:<airtime> VO:<airtime> BK:<airtime>
	6. (Optional) Configure airtime% for explicit STAs in a group. Given airtime% is a percentage of Group airtime% 
		wlanconfig athX addsta <mac_addr> <airtime> <group_name>
	7. Commit changes.
		cfg80211tool athX commitatf 1
	8. Verify the output of following debug commands:
		wlanconfig athX showatfsubgroup
		wlanconfig athX showatftable

### 2.11.SSID-based ATF UCI configuration
	uci add wireless atf-config-ssid
	uci set wireless.@atf-config-ssid[0].device=wifi2
	uci set wireless.@atf-config-ssid[0].command=addssid
	uci set wireless.@atf-config-ssid[0].ssid=atf1
	uci set wireless.@atf-config-ssid[0].airtime=80

### 2.12.STA-based ATF UCI configuration
	uci add wireless atf-config-sta
	uci set wireless.@atf-config-sta[0].device=wifi2
	uci set wireless.@atf-config-sta[0].command=addsta
	uci set wireless.@atf-config-sta[0].macaddr=02:03:04:05:06:07
	uci set wireless.@atf-config-sta[0].airtime=15
	uci set wireless.@atf-config-sta[0].ssid=atf1

### 2.13.Group-based ATF UCI configuration
	uci set wireless.@wifi-iface[0].atfssidgroup='1' 
	uci add wireless atf-config-group
	uci set wireless.@atf-config-group[0].device='wifi0'
	uci set wireless.@atf-config-group[0].command='addgroup'
	uci set wireless.@atf-config-group[0].group='GROUP1'
	uci set wireless.@atf-config-group[0].airtime='80'
	uci add_list wireless.@atf-config-group[0].ssid='atf1'
	uci add_list wireless.@atf-config-group[0].ssid='atf2'

### 2.14.Throughput-based ATF configuration
	uci set wireless.@wifi-iface[3].atf_tput_at='1
	uci add wireless atf-config-tput
	uci set wireless.@atf-config-tput[0].device='wifi2'
	uci set wireless.@atf-config-tput[0].command='addtputsta'
	uci set wireless.@atf-config-tput[0].throughput='25000'
	uci set wireless.@atf-config-tput[0].max_airtime='30'
	uci set wireless.@atf-config-tput[0].macaddr='00:01:02:03:04:05'

### 2.15.AC-based ATF configuration
	uci add wireless atf-config-ac
	uci set wireless.@atf-config-tput[0].device='wifi1'
	uci set wireless.@atf-config-tput[0].command='addac'
	uci set wireless.@atf-config-tput[0].ac=’BE’
	uci set wireless.@atf-config-tput[0].airtime='20'
	uci set wireless.@atf-config-tput[0].ssid= ‘atf1

### 2.16.ATF radio parameters
	1. uci set wireless.wifi2.atfstrictsched='0'—Set to 0 for Fair-queue scheduling. Set to 1 for Strict scheduling.
	2. uci set wireless.wifi2.atfobsssched='0'—Set to 0 to disable OBSS in ATF. Set to 1 to enabled OBSS in ATF. 
	3. uci set wireless.wifi2.atfgrouppolicy='0'—Set to 0 for inter group policy as fair. Set to 1 for inter group policy as strict.

### 2.17.ATF VAP parameters 
	1. uci set wireless.@wifi-iface[1].commitatf=1—Enable or disable ATF on runtime.
	2. uci set wireless.@wifi-iface[1].atfssidsched=1—Set to 1 for Strict scheduling on a specific SSID. Set to 0 for Fair scheduling on a specific SSID. 
	3. uci set wireless.@wifi-iface[3].atf_tput_at='1’—Enable or disable throughput-based ATF.
	4. uci set wireless.@wifi-iface[3].atfssidgroup='1’—Enable or disable ATF SSID grouping.
	5. uci set wireless.@wifi-iface[3].atf_max_buf='512'—Set maximum buffers per peer.
	6. uci set wireless.@wifi-iface[3].atf_min_buf='256'—Set minimum buffers per peer.
	7. uci set wireless.@wifi-iface[3].atfmaxclient='1'—Enable or disable ATF maximum clients feature

## 3.ATF Test Plan
	ap is tx, use iperf -c
	ap tx stas need at the same time as far as possible 
### 3.1.Device Config 
	STA config
	config wifi-device 'wifi1'
			option type 'qcawificfg80211'
			option channel 'auto'
			option macaddr '00:03:7f:12:6a:77'
			option hwmode '11axg'
			option disabled '0'
	
	config wifi-iface
			option device 'wifi1'
			option network 'lan'
			option ssid 'ATF-SSID-0'
			option encryption 'psk2'
			option key '12345678'
			option mode 'sta'
			option wds '1'
	
	
	AP config
	config wifi-device 'wifi0'
			option type 'qcawificfg80211'
			option channel 'auto'
			option macaddr '00:03:7f:87:70:92'
			option hwmode '11axg'
			option disabled '0'
	
	config wifi-iface
			option device 'wifi0'
			option network 'lan'
			option mode 'ap'
			option encryption 'psk2'
			option key '12345678'
			option ssid 'ATF-SSID-0'
			option commitatf '1'
	
	
	config qcawifi 'qcawifi'
			option atf_mode '1'

### 3.2.Test Topology Result
	2.4G ATF work verify:
		AP   : IPQ5018 (11axg mode and 4*4) SSID=ATF-SSID-0
		STA1 : IPQ6018 (mac 00:03:7f:12:6a:77) (IP 192.168.1.103)
		STA2 : IPQ4019 (mac 00:03:7f:12:9a:9a) (IP 192.168.1.200)
	
	1. AP disable ATF function STA1 alone--AP test throughput(AP TX)
		STA1 [  3]  0.0-10.0 sec   137 MBytes   121 Mbits/sec
	
	2. AP enable ATF function STA1 alone--AP test throughput(AP TX)
		STA1 [  3]  0.0-10.1 sec   143 MBytes   119 Mbits/sec
	
	3. AP enable ATF function STA2 alone--AP test throughput(AP TX)
		STA2 [  3]  0.0-10.0 sec   123 MBytes   103 Mbits/sec
	
	4. AP enable ATF function STA2(80%)+STA1(20%)--AP test throughput(AP TX)
		STA1 192.168.1.103  [  3]  0.0-10.0 sec  21.5 MBytes  18.0 Mbits/sec 
		STA2 192.168.1.200  [  3]  0.0-10.1 sec   102 MBytes  84.9 Mbits/sec
	
	5. AP enable ATF function STA2(20%)+STA1(80%)--AP test throughput(AP TX)
		STA1 192.168.1.103  [  3]  0.0-10.0 sec   117 MBytes  97.7 Mbits/sec
		STA2 192.168.1.200  [  3]  0.0-10.0 sec  26.8 MBytes  22.4 Mbits/sec

## 4.ATF Code Implementation

### 4.1.ATF Host and Firmware Design
	Host majority task focus on getting user configuration, allocating table for all SSID, peer and percentage value, 
	calculating each peer entry of ATF, and configuring resources for each peer that need to be send firmware. 
	Host needs to control the interaction between host and firmware. 
	Due to the event of set-up allocation table is triggered by different way either user input or peer join/leave, 
	host generates a timer to do this flow for each event until host passes all configuration resource to firmware via WMI command.

	The host handles all the conversion from the user percentage to per mille (1 part in 1000) distribution across all the clients. without dealing with floating point arithmetic.

	Timer task is split into three sub-tasks, build_atf_alloc_tbl(), cal_atf_alloc_tbl(), and build_atf_for_fm().
		1. Build_atf_alloc_tbl() is responsible for building up the relation between peer and SSID, collecting all peer information and looking up the ATF config VAP table
		2. Cal_atf_alloc_tbl() is based on the relations set up between peers and VAPs. It calculates every VAP and associated peer’s ATF percentage number
		3. Build_atf_for_fm() derives every peer’s MAC address and the corresponding calculated ATF’s number from the table, and builds and passes configuration information to the firmware via WMI interface.

### 4.2.Strict queue and fair queue ATF algorithm
	Airtime Fairness implements two scheduling algorithms which are mutually exclusive: Strict queue and fair queue algorithm. 
	The Strict queue algorithm follows strict airtime allocation as configured by the user and does not try and utilize any unused bandwidth. 
	The fair queue algorithm on the other hand guarantees the configured airtime in congested environments and it also utilizes any unused bandwidth.
	
	For Example:
	In a case where Client 2 is idle and data is sent at peak rate to Client 1: 
	with Strict queue algorithm, client 1 is allotted with only 60% of airtime and the remaining 40% is reserved for Client2. 
	With Fair queue algorithm, Client 1 will try and utilize the unused 40% allotted to Client 2. So in-effect Client 1 will be allotted almost 100% of the airtime as long as Client 2 is idle.

#### 4.2.1.ATF table update scheme for strict queue scheduling algorithm
	Each ‘Token’ is a unit of time and each token equates to 1 microsecond.
	1. After the ATF table is updated following a user configuration or client join/leave, update_atf_nodetable routine iterates through each peer entries in the ATF table and finds the corresponding entry in the node table (struct ieee80211_node).
	2. The routine then reads the ATF unit allotted to the peer (ic->atfcfg_set.peer_id[i].sta_cal_value) and updates this in the node structure.
	3. A timer, atf_tokenalloc_timer configured to run every 200 milliseconds, iterates through the node table. This timer routine converts atf units to tokens that can be passed on to the scheduler. 
	4. For each valid node entry in the node table, the timer reads the atf unit and calls the routine ieee80211_distribute_atf_txtokens to distribute tokens. The tokens are passed on to the LMAC layer. 
	
	Tx tokens = atf_unit * (Timer Interval in micro seconds) / ATF DENOMINATION
	where ATF DENOMINATION is set to 1000. The Choice to use 1000 ( 1 part of 1000) rather than percentage ( 1 part of 100) will help us to avoid dealing with floating point arithmetic and thus less approvimation.
	Thus, for a peer with 100% atf allotted, txtokens are 200000 (1000 * (200 * 1000) /1000 = 200000 usec)

#### 4.2.2.ATF table update scheme for fair-queue scheduling algorithm
	The fair queue algorithm maintains a history/traffic pattern to identify the bandwidth usage of each nodes. 
	This history/pattern is consulted to mark a node as ‘borrow’ enabled or ‘contribute’ enabled. 
	If the node pattern shows up that the average unused tokens is less than a set threshold, the node is marked to ‘borrow’ tokens from other nodes. 
	Similarly if the node pattern shows up that the average unused node tokens are more than a set threshold, the unused tokens will be added up in the contributable token pool which can then be distributed to nodes with borrow enabled. 
	
### 4.3.Restricted fair scheduling
	Restricted fair scheduling allows user to put an upper bound on the amount of airtime that a ATF node can use during fair scheduling to accomplish functionalities .
	
	In this sample illustration, user configures SSID1 (40% Airtime) in regular fair mode and SSID2 (60% Airtime) in restricted fair mode. 
	Peers in SSID1 can use 40% of airtime which is allocated in prior and can salvage airtime from SSID2 if peers in SSID2 does not have any active traffic. 
	SSID1 can reach up to 100% airtime in regular fair mode. However, peers in SSID1 can use only the allocated 40% of its airtime and cannot go beyond 40% of airtime even though SSID2 does not have any active traffic.
	
### 4.4.ATF API
	wlan_atf_init(): API to init airtime fairness component
	wlan_atf_deinit(): API to deinit airtime fairness component
	
	wlan_atf_node_join_leave(): API to indicate node join and leave

	ucfg_atf_set(): API to set airtime fairness enable
	ucfg_atf_get(): API to get airtime fairness enable/disable
	ucfg_atf_clear(): API to set airtime fairness disable

	ucfg_atf_set_txbuf_share(): API to enable/disable txbuf share
	ucfg_atf_get_txbuf_share(): API to get configured txbuf share

	ucfg_atf_set_max_txbufs(): API to set maximum number of tx buffers for atf
	ucfg_atf_get_max_txbufs(): API to get maximum number of tx buffers for atf
	
	ucfg_atf_set_min_txbufs(): API to set minimum number of tx buffers for atf
	ucfg_atf_get_min_txbufs(): API to get minimum number of tx buffers for atf

	ucfg_atf_set_airtime_tput(): API to set atf throughput
	ucfg_atf_get_airtime_tput(): API to get atf throughput

	ucfg_atf_set_max_client(): API to enable/disable atf maxclient feature
	ucfg_atf_get_max_client(): API to get state of maxclient feature

	ucfg_atf_set_ssid_group(): API to enable/disable ssid group for a radio
	ucfg_atf_get_ssid_group(): API to get ssid grouping state for a radio

	ucfg_atf_set_ssid_sched_policy(): API to set a ssid scheduling policy (fairq or strictq)
	ucfg_atf_get_ssid_sched_policy(): API to get scheduled atf policy

	ucfg_atf_set_per_unit(): API is used to set ATF units to default 1000
	ucfg_atf_get_per_unit(): API to get configured ATF units


	ucfg_atf_set_ssid(): API to configure particular SSID
	ucfg_atf_delete_ssid(): API to remove configuration of particular SSID

	ucfg_atf_set_sta(): API to configure particular STA
	ucfg_atf_delete_sta(): API to remove configuration of particular STA
	
	struct wlan_lmac_if_atf_rx_ops //All ATF specific calls from lmac level are done by using Rx operations call back functions 
	struct wlan_lmac_if_atf_tx_ops //All ATF specific calls to lmac level are done by Tx operations call back functions

	For communication from ATF to external modules using external module Public APIs, defined in wlan_atf_utils_api.h

### 4.5 ATF structures
	Group : wal_atf_group 
	VDEV/SSID : wal_atf_vdev
	Peer : wal_atf_peer
	subgroup_list : list of ATF group instances
	atf_ac_config : ATF AC Config

	
### 4.6 WMI commands that are agreed upon between host and firmware to support ATF
	WMI_ATF_SSID_GROUPING_REQUEST_CMDID //ATF SSID GROUPING REQUEST command 
	WMI_VDEV_PARAM_ATF_SSID_SCHED_POLICY //Per ssid (vdev) based ATF strict/fair scheduling policy
	WMI_PEER_ATF_EXT_REQUEST_CMDID //ATF Peer Extended Request command
	WMI_ATF_GROUP_WMM_AC_CONFIG_REQUEST_CMDID //WMM ATF Configuration for groups
	
	
### 4.7.Order of WMI commands from host when ATF mode is enabled
	1. WMI_ATF_SSID_GROUPING_REQUEST_CMDID
		a. Contains number of groups and per group airtime%, scheduling policy 
		b. By default, it creates group0 and assigns 100% of air-time to that group
	2. WMI_ATF_GROUP_WMM_AC_CONFIG_REQUEST_CMDID
		a. Contains number of groups and each groups’ relative airtime% of each AC.
		b. For unconfigured ACs, given airtime% is 0
	3. WMI_PEER_ATF_REQUEST_CMDID
		a. Contains count of all associated peers and each peers’ airtime units. 
		b. In case of implicit peer and valid per AC atf units for a group, peer atf units given in this command may be 0 or invalid.
	4. WMI_PEER_ATF_EXT_REQUEST_CMDID
		a. Contains count of all associated peers and whether each peers’ airtime configured with above command is explicit or implicit.
		b. In case of implicit peer and valid per AC atf units for a group, peer atf units given in previous command may be 0 or invalid.


## 5.ATF Function Need Considered

### 5.1.Other BSS interference
	Airtime distribution between the connected STAs can be accurately distributed, only if interference due to Other BSS are considered. 
	The WLAN driver determines the actual available bandwidth, using the hardware cycle counters.


### 5.2.ATF — Tx buffer distribution
	Apart from token distribution between different clients in the ratio of allotted airtime, we can also distribute the available Tx Buffers between these clients in the same ratio. 
	This can be dynamically enabled/disabled using the cfg80211tool athx atf_shr_buf 1/0 command. 
	This is needed for protocols without flow control, for example, UDP. 
	Otherwise traffic flows for clients having less airtime can potentially hold majority of the Tx Buffers, leading to more packet drops for flows having larger airtime.

### 5.3.ATF — Maximum clients support
	With ATF enabled, maximum clients supporting is limited up to 50.
	With this feature, 32 clients will be part of ATF table (ATF capable clients) and remaining clients outside ATF table will contend for any unassigned airtime.

	For Example:
	Consider an AP configured with 1 VAP, VAP1. The VAP is assigned 80% of airtime through VAP based ATF configuration. 
	If there are 100 clients connected to this AP, the first 32 clients will share 80% of airtime and the remaining 68 clients will contend for 20% of unassigned airtime.

### 5.4.ATF— SSID grouping support
	ATF implementation also supports SSID grouping, where groups can be created, with each groups containing an SSID or multiple SSIDs. 
	Airtime can then be assigned to a group, which is distributed within and across groups. 

#### 5.4.1.Configure SSID Grouping for ATF
	BSSIDs can be grouped together to allow a percentage of airtime to be shared across multiple BSSIDs. 
	Transmit opportunities are allocated based on configurable weight (percentage of airtime). 
	All clients in a BSSID group share the airtime percentage allocated to the BSSID group.

	The following commands are added for ATF SSID grouping configuration:
		1. wlanconfig <vap name> addatfgroup <group name> <ssid> : Creates a group (if it does not exist) and adds ssid to the group created. If the group already exist, adds the ssid to the group
		2. wlanconfig <vap name> configatfgroup <airtime %> : Reserve airtime for a group
		3. wlanconfig <vap name> delatfgroup <group name>: Deletes a group 
		4. wlanconfig <vap name> showatfgroup: Displays the group configured. 
		5. wlanconfig <vap name> showatftable: Displays the group and client-wise airtime allocation. 

### 5.5.ATF – Guaranteed throughput or bandwidth fairness
	When a client connects, first, it receives a share of the unallocated airtime.

	At the next airtime allocation, list of all connected clients is traversed. For those which have a minimum throughput requirement, the achievable UDP throughput is calculated from the phyrate and PER. 
	The throughput can be estimated assuming optimal aggregation and optimal packet size, that is already present in ‘userRateKbps’ field of the rate table. 
	From that the airtime needed to satisfy the minimum throughput requirement is determined. 

	If the client had some airtime allotted to it, and the new airtime requirement is less than or equal to the allotted airtime, then it is configured with its new requirement. 
	Else, the extra airtime needed for it is noted. 
	If the client had some max airtime configured for it and the new requirement is more than that, then the new requirement is limited to the max airtime. 

### 5.6.ATF- per SSID scheduling policy configuration
	ATF implementation also supports per SSID scheduling policy configuration. 
	The configuration change is to mark/configure any SSIDs that should follow strict scheduling policy. 
	By default, all SSIDs will be configured to follow the fair scheduling policy. 

### 5.7.ATF hierarchy distribution
	When a peer MAC is configured with airtime percentage, the ATF percentage applies for that SSID only. 
	If the peer MAC is already configured with ATF percentage for the SSIDs between which it switches, the ATF setting configured for those SSIDs applies. 
	If the peer MAC is not configured with ATF percentage when it switches between SSIDs, then it shares the ATF percentage in the SSID to which it moves, based on the available airtime present in that SSID.
 	When the peer MAC moves back to previous SSID, the previously allocated airtime for the peer MAC in that SSID applies.
	
	Before the implementation of this functionality, peer MAC is allocated airtime from the global ATF percentage. 
	Consider the following network topology with a 2 GHz band that has two SSIDs—HomeSSID with 40% ATM allocation and PublicSSID with 10% ATM allocation. 
	HomeLaptop is allocated 70% of airtime in HomeSSID (that is, 70% x 40% = 28% of overall 2G airtime). 

	When this HomeLaptop moves to PublicSSID, it shares airtime with other clients in 10% of 2G PublicSSID allocation if total airtime for all clients <=10%; else, reassignment of airtime is needed. 
	HomeLaptop has 70% x 10% =7% of overall 2G airtime, assuming no other client is in PublicSSID. 
	However, if HomeLaptop requested 70% and PublicSSID has only 3% remaining, it prompts for reassignment and assigns 0% ATF to the client that moved to the PublicSSID. 
	When the HomeLaptop moves back to HomeSSID, it resumes 28% allocation, assuming no reassignment is configured; else, the changed airtime reassignment value applies in the HomeSSID.

	With the implementation of this functionality, peer MAC is allocated airtime within the hierarchy of the radio and SSID. Consider the following network topology with a 2 GHz band that has two SSIDs—HomeSSID with 40% ATM allocation and PublicSSID with 10% ATM allocation. 
	HomeLaptop is allocated 70% of airtime in HomeSSID (that is, 70% x 40% = 28% of overall 2G airtime). 
	HomeLaptop is allocated 30% of airtime in PublicSSID that is, 30% x 10% = 3% of overall 2G. 

	When this HomeLaptop moves to PublicSSID, it shares airtime with other clients in 10% of 2G PublicSSID allocation based on the configured value. 
	HomeLaptop has 30% x 10% =3% of overall 2G airtime, assuming no other client is in PublicSSID. 
	However, clients might have already used more than 7% ATF that the user configured in PublicSSID. In such a case, user is prompted for reassignment. 
	HomeLaptop is not configured for PublicSSID; instead, it takes the same percentage as configured for HomeSSID to maintain backward compatibility.

### 5.8.WMM AC ATF for groups
	Access Category-based ATF
	AC based configuration will not have any effect on Configured Clients. In other words, all AC’s of a configured client will share the airtime allocated to that specific client.
	Users can create configurations such as the following examples:
	VO (25%), BE (25%), BK (25%), VI (25%)
	VO (30%), BE/BK/VI (70%)

### 5.9.Sample AC-based ATF configuration
	Consider a SSID based configuration with SSID assigned 80% of airtime. AC 2 is configured 10%, AC3 is configured 25% , Peer1 configured with 10% and Peer 2 configured with 20%. Peer3 and Peer 4 are unconfigured clients. All Peers are connected to the same SSID. 
		1.Peer1 will be guaranteed 8% of airtime (10% of 80). All ACs of Peer1 will share 8%. 
		2.Peer2 will be guaranteed 16% (20% of 80%). All ACs of Peer2 will share 16%
		3.Residual airtime is 56% (80 – (8 +16))
		AC0-BE AC1-BK AC2-VI AC3-VO

		4.AC2 of Peer 3 and 4 will be guaranteed 10% of unconfigured/Residual airtime, 5.6%. residual airtime is 56% (80 – (8 +16)). 
		5.AC3 of Peer 3 and 4 will be guaranteed 25% of unconfigured/Residual airtime, 14%.		
		6.AC0, AC1 of Peer 3and 4 will be guaranteed the rest 36.4%. Residual airtime is 56%

### 5.10.ATF fair scheduling with idle clients
	Support is implemented to identify the idle clients in the AP network and distribute their unused ATF tokens to the active clients.

### 5.11.TCP flow control influence
	TCP traffic increases gradually to behave according to the requirement of this feature, since the TCP window in IP layers also does the flow control.