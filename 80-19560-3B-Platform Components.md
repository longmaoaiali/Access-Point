### Host -- Firmware SRNG interface
	Host SRNG interface to Control rings(WMI,HIT ...)
	Data rings(TCL,REO,WBM ...)

### QMI (Qualcomm messaging interface)
	CNSS Daemon service communicate with firmware through QMI interface. 
	Control messages between WLAN host driver and Q6 firmware are exchanged over QMI.

	Host Wi-Fi driver executes on ARM processor and WLAN firmware on Q6. 
	While Q6 and A53 have multiple communication channels such as copy engines and Data DMA channels, these channels are used for data/WMI communication.

### Purpose of QMI messages
	 Provide copy engine configuration to target firmware.
	 Firmware can request host memory for its purpose. For multiple reasons, firmware would need memory at host. Examples are Code swap regions (dlpager), peer cache. 
	 Identification of board type for calibration, BDF download. Firmware advertises the board type (aka card type) through one of the QMI messages to host. Host would use board_id provided to download appropriate board data file.
	 Enable appropriate mode of operation (such as FTM, Mission mode, Coldboot Calibration)
	 Inform firmware to enable some of the services through its announcement of host capabilities.
	 Provide host memory region information for target QDSS for QDSS logging and log data.

### MHI(Modem Host Interface) for PCI radio FW download procedure
	FW is downloaded with the help of MHI. MHI uses Copy engine 9, 10, and 11 for its operation. 
	During the FW load, the MHI module allocates multiple segments of 512K blocks of physically contiguous memory and copies the FW code/data in the allocated segments. First the SBL gets downloaded and executed and then the full firmware gets downloaded and then it gets executed. 
	Refer mhi_fw_load_worker() in the MHI driver.

### IRQ mapping
	The QCN90xx/QCN602x/QCN92xx currently uses 16 MSI interrupts to operate per radio.
	Its uses 3 MSI for MHI operation, 5 interrupts for Copy Engine control operation and 8 interrupts for Data path. 
	The DP interrupts are shared between multiple rings to accommodate all the rings within the available 16 interrupts.

	IPQ807xA/IPQ817x/IPQ60xx/IPQ95xx IRQ Mapping
	Each copy engine ring is provided a separate GIC interrupt line and each data path ring is also provided a separate GIC interrupt line. 
	MSI interrupts is not used in the IPQ807xA/IPQ817x or IPQ60xx. See the corresponding device .dts file for exact assignment.

### athdiag tool to read/write the memory addresses 
	athdiag --get --address <address>
	athdiag --set --address <address> --value <value>

### cnsscli
	cnsscli is a user interface application that provides basic commands to enable and disable QDSS Trace Dump collection for each radio. 
	It communicates over datagram socket to send commands to cnss-daemon.

	cnsscli commands to enable and disable QDSS tracing
	To start QDSS tracing:
		cnsscli -i <interface> --qdss_trace_start
	To stop QDSS tracing:
		cnsscli -i <interface> --qdss_trace_stop 0x3f --> For QCN90xx/QCN60xx
		cnsscli -i <interface> --qdss_trace_stop 1 --> For QCN92xx

### Parsing of m3 dump 
	 When this feature is enabled and m3 crash happens m3 dump will collect to tftp use following steps to parse m3 dump bin.
	 Run the script parse_m3_dump_pine.sh or parse_m3_dump_cyp.sh to get phydbg and pdmem log.
	 Script location: lithium_crash_parser_master\tools\hardwareRelatedTools\phy_tracer\pine\arch\phy\lithium\src\digital\e2e\fw\scripts\phydbg_analysis
	 Requirements – Linux machine with python 3.4 and above
	 Command – parse_m3_dump_pine.sh m3_dump_qcnxxxx_*.bin (for QCN90xx/QCN602x/QCN92xx) and parse_m3_dump_cyp.sh m3_dump_*.bin (for IPQ60xx)
	 The preceding command gives two files: pdmem.bin and phydbg.txt
	 Pdmem.bin can be directly used by ucode team to analyze the ucode crash root cause
	 Phydbg.txt can be used as input file to existing parsing method to get parsed excel phydbg logs. 
	ie - parse.py --xlsx_fn <phy.xlsx> --target Pine <phydbg.txt> or - parse.py --target Cypress --xlsx_fn <phy.xlsx> <phydbg.txt> from lithium_crash_parser_master\tools\hardwareRelatedTools\phy_tracer\pine\arch\phy\lithium\src\digital\e2e\fw\scripts\phydbg_analysis

### Ramdump collection
	PCI radio Ramdump collection
		With the list of segments provided to do_elf_ramdump(), the function would create a /dev/ramdump_QCNXXXX_PCIx (x= PCI slot number i.e. 0 or 1) and make the debug data available as an elf file to the user file system.

	Integrated radio Ramdump collection
		The q6v5 wcss rproc driver creates device file with class name “dump” and the device file would appear in the name /dev/q6mem
		memory-region = <&q6_region>, <&q6_etr_region>;
		
	 IPQ50xx and QCN61xx ramdump collection in MultiPD model
