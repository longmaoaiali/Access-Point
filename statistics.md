### WLAN data path statistics
	cfg80211tool ath0 txrx_stats 258 Rx rate statistics
	cfg80211tool ath0 txrx_stats 259 Tx rate statistics
	cfg80211tool ath0 txrx_stats 260 Rx PDEV statistics
	cfg80211tool ath0 txrx_stats 261 Tx PDEV statistics
	cfg80211tool ath0 txrx_stats 257 Clear statistics
	cfg80211tool wifi0 fc_peer_stats <mac_addr> Host peer statistics
	cfg80211tool wifi0 dp_fw_peer_stats <mac_addr>:0x3 Firmware peer statistics
	cfg80211tool wifi0 txrx_stats <1-16> Firmware statistics

### Radio-specific statistics
	WIFILI_RX_MSDU_ERROR Rx MSDU errors
	WIFILI_RX_INV_PEER_RCV Invalid peers received
	WIFILI_RX_WDS_SRCPORT_EXCEPTION WDS source ports learned for WDS entries
	WIFILI_RX_WDS_SRCPORT_EXCEPTION_FAIL WDS source port exception learn failures
	WIFILI_RX_DELIVERD Delivered Rx 
	WIFILI_RX_DELIVER_DROPPED Dropped Rx delivers 
	WIFILI_RX_INTRA_BSS_UCAST Received IBSS unicast packets
	WIFILI_RX_INTRA_BSS_UCAST_FAIL Failed IBSS unicast packets received
	WIFILI_RX_INTRA_BSS_MCAST MC or BC packets received
	WIFILI_RX_INTRA_BSS_MCAST_FAIL Failed MC or UC packets received
	WIFILI_RX_SG_RCV_SEND Scatter-gather frames received
	WIFILI_RX_SG_RCV_FAIL Failed scatter-gather frames received
	WIFILI_RX_MCAST_ECHO Multicast echo packets received
	WIFILI_TX_ENQUEUE Packets enqueued to software queue to send to hardware
	WIFILI_TX_ENQUEUE_DROP Failed packets to enqueue to software queue
	WIFILI_TX_DEQUEUE Packets dequeued from software to queue to hardware ring
	WIFILI_TX_HW_ENQUEUE_FAIL Failed packets to enqueue to hardware
	WIFILI_TX_SENT_COUNT Packets successfully queued to hardware

### Tx hardware ring statistics
	TCL ring statistics
		WIFILI_TCL_NO_HW_DESC Available hardware descriptors
		WIFILI_TCL_RING_FULL Times TCL ring is found to be full while queueing packets to TCL ring
		WIFILI_TCL_RING_SENT Packets successfully queued to TCL ring

	Tx completion ring statistics
		WIFILI_TX_DESC_FREE_INV_BUFSRC Tx completion packets received with invalid buffer source
		WIFILI_TX_DESC_FREE_INV_COOKIE Tx completion packets received with invalid software cookie
		WIFILI_TX_DESC_FREE_HW_RING_EMPTY Tx rings found to be empty while reaping
		WIFILI_TX_DESC_FREE_REAPED Tx completions successfully reaped

### Rx ring statistics
	REO ring statistics
		WIFILI_REO_ERROR Rx REO ring errors
		WIFILI_REO_REAPED Packets successfully reaped from the REO ring
		WIFILI_REO_INV_COOKIE Packets received with invalid cookies

	Rx DMA ring statistics
		WIFILI_RXDMA_HW_DESC_UNAVAILABLE Hardware descriptors available to fill with the packets
		WIFILI_RXDMA_BUF_REPLENISHED Rx buffers added to DMA ring for MAC to fill