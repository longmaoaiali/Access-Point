# Debug Skills


## 1.hostapd/wpa_supplicant debug
	
#### 1.1.wpa_supplicant/hostapd Manual start
    1. killall wpa_supplicant
    2. wpa_supplicant -g /var/run/wpa_supplicantglobal -dddd -B -P /var/run/wpa_supplicant-global.pid /var/run/wpa_supplicant-ath0.conf

#### 1.2.wpa_supplicant/hostapd log 
    modifty /etc/init.d/qca-wpa-supplicant start Shell as below
    wpa_supplicant -g /var/run/wpa_supplicantglobal -dddd  -B -P /var/run/wpa_supplicant-global.pid -f /var/wap.log
    // -dddd improve log level, log saved in /var/wap.log





## 2.Driver Log
	Driver LogLevel set
	cfg80211tool ath0 qdf_cv_lvl 0x008f0003   //set module 008f logLevel 0003
	cfg80211tool ath0 g_qdf_cv_lvl   //get all module logLevel
