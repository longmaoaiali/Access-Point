# System Build


## 2.module build
	tcpdump bin :
		1. Add Cross-compile path to environment PATH
    	export PATH=/home/@@/code_SPF11_3/qca-networking-2020-spf-11-3_qca_oem/qsdk/staging_dir/toolchain-arm_cortex-a7_gcc-5.2.0_musl-1.1.16_eabi/bin:$PATH
		2. Cd generated source Dir
    	cd qsdk/build_dir/target-arm_cortex-a7_musl-1.1.16_eabi/tcpdump-full/tcpdump-4.9.2
  		3. Before make you need make clean first 
  		
		QSDK Configure
		package-$(CONFIG_PACKAGE_tcpdump) += network/utils/tcpdump
		qsdk/.config     CONFIG_DEFAULT_tcpdump=y


	qca-wifi : make package/feeds/qca/qca-wifi/compile V=s
	qca-hostap : make package/feeds/qca/qca-hostap/compile V=s