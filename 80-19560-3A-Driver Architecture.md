### Init

#### Data structure 
	struct ol_ath_soc_softc //SoC data structure
	struct ol_ath_softc_net80211 //PDEV data structure
	struct ol_ath_vap_net80211 //VDEV data structure 

#### Configuration
	# HW mode configuration
	# hw_mode_id 1 – DBDC
	# hw_mode_id 0 – Single radio

#### Function
	__ol_ath_attach //Probe sequence
	__ol_ath_detach //Remove sequence


### LMAC If Interface
	LMAC --> UMAC
	wlan_lmac_if_tx_ops declared in cmn_dev/umac/global_umac_dispatcher/lmac_if/inc/wlan_lmac_if_def.h
	a. target_if_register_tx_ops – For componentized features parallel to UMAC. Example: spectral 
	b. target_if_register_umac_tx_ops() – For componentized UMAC components. Example: ATF and scan.
	c. olif_register_umac_tx_ops() – For non-componentized legacy features. Example: Mgmt TXRX

	UMAC --> LAMC
	a. wlan_global_lmac_if_rx_ops_register() – For componentized features parallel to UMAC. Example: spectral
	b. wlan_lmac_if_umac_rx_ops_register() – For all UMAC componentized features. Example: scan, ATF.

### MGMT TX RX
	wlan_mgmt_txrx_mgmt_frame_tx() //manager frame tx
	wlan_mgmt_txrx_beacon_frame_tx() //beacon tx

	wlan_mgmt_txrx_register_rx_cb() //APIs to register for Mgmt frame reception



### INI File config
	cfg_ini_parse() //parse an ini file
	#TWT Enable/Disable
	twt_enable
	
### tigger target assert
	 cfg80211tool wifiX set_fw_hang <value>

### DSCP TID WMM
	ieee80211_dscp_override() // is called to identify a TID for a specific packet based on its dscp value.
	cfg80211tool wifix s_dscp_tid_map <dscp> <tid>  
	//To configure a specific tid mapping table for a VAP, use the s_dscp_tid_map option. cfg80211tool option set_dscp_ovride must be set to 1.

### Transmit power control per SSID for control frames
	cfg80211tool athX s_txpow frame_subtype
	cfg80211tool athX g_txpow frame_subtype
	For example, to set the transmit power of beacon, enter the following:
		cfg80211tool ath0 s_txpow 0x04 8
	For example, to obtain the Tx power set for beacon, enter the following command:
		cfg80211tool ath0 g_txpow 0x04 wifi0
		get_txpow_ctl:8

### Modify Tx chain-mask without reconnecting clients
	cfg80211tool wifiX txchainsoft 0xN
	Increasing or decreasing the Tx chain appropriately impacts the NSS count listed under Firmware stats 6 section of the txrx_fw_stats command
	cfg80211tool ath1 txrx_fw_stats 6 //TX RX stats

### Green AP
	Green AP feature when enabled, powers of some of the antennas when no station is connected to any of the VAPs in the radio. Green AP context is initialized at dev_attach().  

	10.19 Greenfield operation
	An HT STA shall not transmit a frame with the TXVECTOR parameter FORMAT set to HT_GF unless the RA of the frame corresponds to a STA for which the HT-Greenfield subfield of the HT Capabilities element contained a value of 1 and dot11HTGreenfieldOptionActivated is true.
	WMI Command : WMI_PDEV_GREEN_AP_PS_ENABLE_ CMDID 
	wlan_green_ap_start()

	10.26.3 Protection mechanisms for transmissions of HT PPDUs
	Transmissions of HT PPDUs, referred to as HT transmissions, are protected if there are other STAs present that cannot interpret HT transmissions correctly. 
	The HT Protection and Nongreenfield HT STAs Present fields in the HT Operation element within Beacon and Probe Response frames are used to indicate the protection requirements for HT transmissions.

### Set TX queue parameters
	.set_txq_params = wlan_cfg80211_set_txq_params,

### Scanning operation
	.scan = wlan_cfg80211_scan_start, // Scan request operation 

	Indicating scan done / scan results to upper layers
		cfg80211_scan_done
		cfg80211_inform_bss
		cfg80211_inform_bss_frame
		
### Hostapd -- Cfg80211 MAC/PHY Capabilities
	NL80211_BAND_IFTYPE_ATTR_HE_CAP_MAC  
	NL80211_BAND_IFTYPE_ATTR_EHT_CAP_MAC: EHT MAC capabilities as in EHT capabilities IE
	NL80211_BAND_IFTYPE_ATTR_EHT_CAP_MCS_SET

### 11BE mode channel defination
	static const struct bonded_channel_freq bonded_chan_160mhz_list_freq[] {
		{5180, 5320},
		{5500, 5640},
		{5745, 5885},
	#ifdef CONFIG_BAND_6GHZ
		{5955, 6095},
		{6115, 6255},
		{6275, 6415},
		{6435, 6575},
		{6595, 6735},
		{6755, 6895},
		{6915, 7055}
	#endif /*CONFIG_BAND_6GHZ*/
	}

### 802.1d 
	802.11d is a wireless specification for operation in additional regulatory domains. 

### MU-EDCA 
	IEEE 802.11e will provide Quality of Service (QoS) support in Wireless Local Area Network (WLAN). 
	802.11e - Wi-Fi Multimedia (WMM)

	enhanced distributed channel access (EDCA)  
	On the AP side, the MU EDCA parameters are configured. When MU EDCA is enabled on the AP, the MU EDCA IE is added to the beacons and the probe response frames.
	On the STA side, the STA parses the MU EDCA IEs and update its MU EDCA parameter values with the ones advertised by the AP.

	struct muedcaParams{}
	cfg80211tool athX set_muedcaparams

EDCA Txops, access category(BE)， CW,random back off
HW/MAC generated a random backoff (CW) within CWmin and CWmax values and transmission was performed for TxOP.
5.9 TxOPs of 10 ms for best effort access category

###Tx sequence completion ISR
	In the Tx sequence completion interrupt service routine (ISR), the last TxOP time for the transmitted packet is accumulated until the burst sequence is completed. The average TxOP is computed and stored in the wal_txq structure. Tav = 0.9*Tav + 0.1* current TxOP

###Tx send post-PPDU
	In Tx send module, before PPDU Posting , the Avg TxOP in the Best Effort txq is checked and if it is > 6ms, the duty cycle for CWmin15, CWmin31 , CWmax31 and CWmax63 is computed as per the System team Suggesting. 
	This computation is done for every 100ms interval and CWmin, CWmax value for the Best Effort TxQ is changed accordingly to the duty cycle.
	When the average TxOP is less than 6ms, the default CW (min,max) of (15,63) is restored.

	European regulations enable up to 10 ms TxOPs for APs operating in the 5 GHz band. 
	However, TxOPs longer than 6 ms must be compensated using a double random range on the backoff that follows the long TxOP. The CW sequence effectively becomes 31/31/63 for an AP, instead of 15/31/63. This is referred to as CW doubling.

###Medium Access Testing
	For medium access, the test verifies the distribution of the time between TxOPs (backoff) when there are no collisions. 
	The backoffs must be distributed uniformly between 41 us and 176 us for TxOPs < 6 ms and between 41 us and 320 us for TxOPs > 6 ms (and < 10 ms).
	41 us = AIFS - 2 us = 16 + 3*9 - 2
	176 us = AIFS + 15 slots - 2 us = 16 + 3*9 + 15*9 - 2 us
	320 us = AIFS + 31 slots - 2 us = 16 + 3*9 + 31*9 - 2 us
	2 us is subtracted to allow room for measurement and implementation errors