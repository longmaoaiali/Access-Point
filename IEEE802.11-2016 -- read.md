# IEEE802.11-2016

#### Content Overview

	Directional multi-gigabit (DMG) PHY specification for 60GHz
	The 11ad physical layer was originally added as an amendment, and now is included as chapter 20 of the 802.11-2016 standard. It is called "Directional Multi-Gigabit (DMG) PHY".

	Television Very High Throughput (TVHT) PHY specification for 54 and 790 MHz
	The 802.11af amendment defines enhancements to the 802.11 WLAN physical layer (PHY) and medium access control (MAC) specifications to support operation in the TV white space (TVWS) spectrum in the VHF and UHF bands between 54 and 790 MHz. 802.11af introduces a new Television Very High Throughput (TVHT) PHY to accommodate the narrow TV channels that are available in the TVWS spectrum.

#### Definition
	aggregate medium access control (MAC) protocol data unit (A-MPDU): A structure that contains one or more MPDUs and is transported by a physical layer (PHY) as a single PHY service data unit (PSDU).
	
	aggregate medium access control (MAC) service data unit (A-MSDU): A structure that contains one or more MSDUs and is transported within a single (unfragmented) data medium access control (MAC) protocol data unit (MPDU).

	basic service set (BSS) max idle period: A time period during which the access point (AP) does not disassociate a station (STA) due to nonreceipt of frames from that STA. 

	basic service set (BSS) transition: Change of association by a station (STA) from one BSS to another BSS in the same extended service set (ESS).

	beamformee: A station (STA) that receives a physical layer (PHY) protocol data unit (PPDU) that was transmitted using a beamforming steering matrix. RX
	
	beamformer: A station (STA) that transmits a physical layer (PHY) protocol data unit (PPDU) using a beamforming steering matrix. TX

	beamforming: A spatial filtering mechanism used at a transmitter to improve the received signal power or signal-to-noise ratio (SNR) at an intended receiver. Syn: beam steering. 

	beamforming steering matrix: A matrix determined using knowledge of the channel between a transmitter and an intended receiver that maps from space-time streams to transmit antennas with the goal of improving the signal power or signal-to-noise ratio (SNR) at the intended receiver.

	cipher suite: A set of one or more algorithms, designed to provide data confidentiality, data authenticity or integrity, and/or replay protection.

	clear channel assessment (CCA) function: The logical function in the physical layer (PHY) that determines the current state of use of the wireless medium (WM).

	time unit (TU): A measurement of time equal to 1024 µs.

	traffic category (TC): A label for medium access control (MAC) service data units (MSDUs) that have a distinct user priority (UP), as viewed by higher layer entities, relative to other MSDUs provided for delivery over the same link. Traffic categories are meaningful only to MAC entities that support quality of service (QoS) within the MAC data service. 

	traffic identifier (TID): Any of the identifiers usable by higher layer entities to distinguish medium access control (MAC) service data units (MSDUs) to MAC entities that support quality of service (QoS) within the MAC data service. 

	traffic stream (TS): A set of medium access control (MAC) service data units (MSDUs) to be delivered subject to the quality-of-service (QoS) parameter values provided to the MAC in a particular traffic specification  (TSPEC). TSs are meaningful only to MAC entities that support QoS within the MAC data service. 

	transmission opportunity (TXOP): An interval of time during which a particular quality-of-service (QoS) station (STA) has the right to initiate frame exchange sequences onto the wireless medium (WM). 

	transmit chain: The physical entity that implements any necessary signal processing to generate the transmit signal from the digital baseband. Such signal processing includes digital to analog conversion, filtering, amplification and up-conversion

	wildcard BSSID: A BSSID value used to represent all BSSIDs.
	wildcard SSID: A SSID value used to represent all SSIDs.

	Clause 19 specifies the PHY entity for a high-throughput (HT) orthogonal frequency division multiplexing (OFDM) system. 
		— Clause 16 for HR/DSSS
		— Clause 17 for OFDM
		— Clause 18 for ERP
		— Clause 19 for HT
		— Clause 20 for DMG
		— Clause 21 for VHT
		— Clause 22 for TVHT

	4-way handshake: A pairwise key management protocol defined by this standard. This handshake confirms mutual possession of a pairwise master key (PMK) by two parties and distributes a group temporal key(GTK).

	
	20 MHz mask physical layer (PHY) protocol data unit (PPDU): One of the following PPDUs: 
		a) A Clause 17 PPDU transmitted using the 20 MHz transmit spectral mask defined in Clause 17.
		b) A Clause 18 orthogonal frequency division multiplexing (OFDM) PPDU transmitted using the transmit spectral mask defined in Clause 18.
		c) A high-throughput (HT) PPDU with the TXVECTOR parameter CH_BANDWIDTH equal to HT_CBW20 and the CH_OFFSET parameter equal to CH_OFF_20 transmitted using the 20 MHz transmit spectral mask defined in Clause 19.
		d) A very high throughput (VHT) PPDU with TXVECTOR parameter CH_BANDWIDTH equal to CBW20 transmitted using the 20 MHz transmit spectral mask defined in Clause 21.
		e) A Clause 17 PPDU transmitted by a VHT STA using the transmit spectral mask defined in Clause 21.
		f) An HT PPDU with the TXVECTOR parameter CH_BANDWIDTH equal to HT_CBW20 and the CH_OFFSET parameter equal to CH_OFF_20 transmitted by a VHT STA using the 20 MHz transmit spectral mask defined in Clause 21.
		// TXVECTOR is introducted in HT Phy
	
	access category (AC): A label for the common set of enhanced distributed channel access (EDCA) parameters that are used by a quality-of-service (QoS) station (STA) to contend for the channel in order to transmit medium access control (MAC) service data units (MSDUs) with certain priorities.

	authentication and key management (AKM) suite: A set of one or more algorithms designed to provide authentication and key management, either individually or in combination with higher layer authentication and key management algorithms outside the scope of this standard.

	delivery traffic indication map (DTIM) interval: The interval between the consecutive target beacon transmission times (TBTTs) of beacons containing a DTIM. The value, expressed in time units, is equal to the product of the value in the Beacon Interval field and the value in the DTIM Period subfield in the TIM element in Beacon frames

	downlink: A unidirectional link from an access point (AP) to one or more non-AP stations (STAs) 

	downlink multi-user multiple input, multiple output (DL-MU-MIMO): A technique by which an access point (AP) with more than one antenna transmits a physical layer (PHY) protocol data unit (PPDU) to multiple receiving non-AP stations (STAs) over the same radio frequencies, wherein each non-AP STA simultaneously receives one or more distinct space-time streams.
	
	dynamic bandwidth operation: A feature of a very high throughput (VHT) station (STA) in which the request-to-send/clear-to-send (RTS/CTS) exchange, using non-high-throughput (non-HT) duplicate physical layer (PHY) protocol data units (PPDUs), negotiates a potentially reduced channel width (compared to the channel width indicated by the RTS) for subsequent transmissions within the current transmission opportunity (TXOP).

	EAPOL-Key confirmation key (KCK): A key used to integrity-check an EAPOL-Key frame.
	EAPOL-Key encryption key (KEK): A key used to encrypt the Key Data field in an EAPOL-Key frame.
	EAPOL-Key frame: A Data frame that carries all or part of an IEEE 802.1X EAPOL PDU of type EAPOL-Key. 
	EAPOL-Key request frame: A Data frame that carries all of part of an IEEE 802.1X EAPOL-Key protocol data unit (PDU) with the Request bit in the Key Information field in the IEEE 802.11 key descriptor set to 1.
	EAPOL-Start frame: A Data frame that carries all or part of an IEEE 802.1X EAPOL PDU of type EAPOL-Start.

	enhanced distributed channel access (EDCA): The prioritized carrier sense multiple access with collision avoidance (CSMA/CA) access mechanism used by quality-of-service (QoS) stations (STAs) in a QoS basic service set (BSS) and STAs operating outside the context of a BSS.
	//a kind of CSMA/CA for Qos BSS 

	extended channel switching (ECS): A procedure that is used to announce a pending change of operating channel, operating class, or both.
	extended rate physical layer (ERP): A physical layer (PHY) compliant with Clause 18

	fast basic service set (BSS) transition (FT) 4-way handshake: A pairwise key management protocol used during FT initial mobility domain association. This handshake confirms mutual possession of a pairwise master key, the PMK-R1, by two parties and distributes a group temporal key (GTK).

	fast basic service set (BSS) transition (FT) initial mobility domain association: The first association or first reassociation procedure within a mobility domain, during which a station (STA) indicates its intention to use the FT procedures.

	fast basic service set (BSS) transition (FT) originator: A station (STA) that initiates the FT protocol by sending an FT Request frame or an Authentication frame with Authentication Algorithm Number field equal to Fast BSS Transition.

	fragmentation: The process of partitioning a medium access control (MAC) service data unit (MSDU) or MAC management protocol data unit (MMPDU) into a sequence of smaller MAC protocol data units (MPDUs) prior to transmission. The process of recombining a set of fragment MPDUs into an MSDU or MMPDU is known as defragmentation. These processes are described in 5.8.1.9 of ISO/IEC 7498-1:1994.
	// MSDU <--> MPDUs  

	group key handshake: A group key management protocol defined by this standard. It is used only to issue a new group temporal key (GTK) to peers with whom the local station (STA) has already formed security associations.

	high-throughput (HT) delayed (HT-delayed) block acknowledgment (Ack): A delayed block ack mechanism that requires the use of the compressed BlockAck frame and the No Acknowledgment ack policy setting within both BlockAckReq and BlockAck frames. 

	high-throughput (HT) immediate (HT-immediate) block acknowledgment (Ack): An immediate block ack mechanism that requires the use of the compressed BlockAck frame and an implicit block ack request and allows partial-state operation at the recipient. 

	high-throughput (HT) greenfield (HT-greenfield) format: A physical layer (PHY) protocol data unit (PPDU) format of the HT PHY using the HT-greenfield format preamble. This format is represented at the PHY data service access point (SAP) by the TXVECTOR/RXVECTOR FORMAT parameter being equal to HT_GF.

	high-throughput (HT) mixed (HT-mixed) format: A physical layer (PHY) protocol data unit (PPDU) format of the HT PHY using the HT-mixed format preamble. This format is represented at the PHY data service access point (SAP) by the TXVECTOR/RXVECTOR FORMAT parameter being equal to HT_MF.

	— Non-HT format (NON_HT): Packets of this format are structured according to the Clause 17 (OFDM) or Clause 18 (ERP) specification. Support for non-HT format is mandatory.
	— HT-mixed format (HT_MF): Packets of this format contain a preamble compatible with Clause 17 and Clause 18 receivers. The non-HT-STF (L-STF), the non-HT-LTF (L-LTF), and the non-HT SIGNAL field (L-SIG) are defined so they can be decoded by non-HT Clause 17 and Clause 18 STAs. The rest of the packet cannot be decoded by Clause 17 or Clause 18 STAs. Support for HT-mixed format is mandatory.
	— HT-greenfield format (HT_GF): HT packets of this format do not contain a non-HT compatible part. Support for HT-greenfield format is optional.


	multi-user (MU) physical layer (PHY) protocol data unit (PPDU): A PPDU that carries one or more PHY service data units (PSDUs) for one or more stations (STAs) using the downlink multi-user multiple input, multiple output (DL-MU-MIMO) technique

	no acknowledgment/no retry (No-Ack/No-Retry): A retransmission policy for group addressed frames in which each frame is transmitted once and without acknowledgment. 

	non-high-throughput (non-HT) physical layer (PHY) protocol data unit (PPDU): A PPDU that is transmitted by Clause 15, Clause 16, Clause 17, or Clause 18 PHY, or not using a TXVECTOR FORMAT parameter equal to HT_MF, HT_GF or VHT

	non-high-throughput (non-HT) SIGNAL field (L-SIG) transmit opportunity (TXOP) protection: A protection mechanism in which protection is established by the non-HT SIG Length and Rate fields indicating a duration that is longer than the duration of the PPDU itself.

	pairwise master key (PMK): The key derived from a key generated by an Extensible Authentication Protocol (EAP) method or obtained directly from a preshared key (PSK).
	//sometime PMK = PSK

	pairwise master key (PMK) R1 key holder (R1KH): The component of robust security network association (RSNA) key management of the Authenticator that receives a PMK-R1 from the R0KH, holds the PMK-R1, and derives the pairwise transient keys (PTKs).

	pairwise master key (PMK) S0 key holder (S0KH): The component of robust security network association (RSNA) key management of the Supplicant that derives and holds the PMK-R0, derives the PMK-R1s, and provides the PMK-R1s to the S1KH

	pairwise master key (PMK) S1 key holder (S1KH): The component of robust security network association (RSNA) key management in the Supplicant that receives a PMK-R1 from the S0KH, holds the PMK-R1, and derives the pairwise transient keys (PTKs).

	pairwise master key security association (PMKSA): The context resulting from a successful IEEE802.1X authentication exchange between the peer and Authentication Server (AS) or from a preshared key(PSK).

	pairwise transient key (PTK): A concatenation of session keys derived from the pairwise master key (PMK) or from the PMK-R1. Its components are a key confirmation key (KCK), a key encryption key(KEK), and a temporal key (TK), which is used to protect information exchanged over the link.

	temporal encryption key: The portion of a pairwise transient key (PTK) or group temporal key (GTK) used directly or indirectly to encrypt data in medium access control (MAC) protocol data units (MPDUs).

	transmitted basic service set identifier (BSSID): The BSSID included in the medium access control (MAC) header transmitter address field of a Beacon frame when the multiple BSSID capability is supported.

	user: An individual station or group of stations (STAs) identified by a single receive address (RA) in the context of single-user multiple input, multiple output (SU-MIMO) or multi-user multiple input, multiple output (MU-MIMO).

#### QoS BSS
	