# Packet Processing API

### `struct sk_buff`
#### `skb_pull(struct sk_buff *skb, unsigned int len)`
	/**
	 *	skb_pull - remove data from the start of a buffer
	 *	@skb: buffer to use
	 *	@len: amount of data to remove
	 *
	 *	This function removes data from the start of a buffer, returning
	 *	the memory to the headroom. A pointer to the next data in the buffer
	 *	is returned. Once the data has been pulled future pushes will overwrite
	 *	the old data.
	 */
	unsigned char *skb_pull(struct sk_buff *skb, unsigned int len)
	{
		return skb_pull_inline(skb, len);
	}

----------
### `struct qdf_nbuf_t` (Network buf instance)

#### `qdf_nbuf_pull_head(qdf_nbuf_t buf, qdf_size_t size)`
	static inline uint8_t *qdf_nbuf_pull_head(qdf_nbuf_t buf, qdf_size_t size) {}
	// pull data out from the front 

	For Example
	static void dp_tx_remove_vlan_tag(struct dp_vdev *vdev, qdf_nbuf_t nbuf)
	{
			struct vlan_ethhdr veth_hdr;
			struct vlan_ethhdr *veh = (struct vlan_ethhdr *)nbuf->data;
		
			qdf_debug("dp_tx_remove_vlan_tag 0x%x",veh->h_vlan_TCI);
			/*
			 * Extract VLAN header of 4 bytes:
			 * Frame Format : {dst_addr[6], src_addr[6], 802.1Q header[4], EtherType[2], Payload}
			 * Before Removal : xx xx xx xx xx xx xx xx xx xx xx xx 81 00 00 02 08 00 45 00 00...
			 * After Removal  : xx xx xx xx xx xx xx xx xx xx xx xx 08 00 45 00 00...
			 */
			qdf_mem_copy(&veth_hdr, veh, sizeof(veth_hdr));
			qdf_nbuf_pull_head(nbuf, ETHERTYPE_VLAN_LEN);
			/* xx xx xx xx xx xx xx xx 81 00 00 02 08 00 45 00 00... */
			veh = (struct vlan_ethhdr *)nbuf->data;
			qdf_mem_copy(veh, &veth_hdr, 2 * QDF_MAC_ADDR_SIZE);
			/* xx xx xx xx xx xx xx xx xx xx xx xx 08 00 45 00 00 */
			return;
	}

	Base Implement: unsigned char *skb_pull(struct sk_buff *skb, unsigned int len) 
	//remove data from the start of a buffer

#### `qdf_nbuf_trim_tail(qdf_nbuf_t buf, qdf_size_t size)`
	static inline void qdf_nbuf_trim_tail(qdf_nbuf_t buf, qdf_size_t size)
	//trim data out from the end
	
	For example:
	static QDF_STATUS tkip_demic(struct wlan_crypto_key *key, qdf_nbuf_t wbuf, uint8_t tid, uint8_t hdrlen)
	{
		struct wlan_crypto_cipher *cipher_table;
		cipher_table = key->cipher_table;
		qdf_nbuf_trim_tail(wbuf, cipher_table->miclen);
		return QDF_STATUS_SUCCESS;
	}
	