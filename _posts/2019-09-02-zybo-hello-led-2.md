---
title: zybo hello led - 2
layout: posts
author_profile: true
read_time: true
comments: null
share: true
related: true
---

여러 이미지들이 빌드되지만 우선 사용되는것은 BOOT.BIN + Image.ub  

BOOT.BIN = zynq fsbl + fpga bitstream + application (u-boot) 일차적인 부트 이미지  
Image.ub = flatteded image tree (FIT)  

또한 BOOT.BIN이 생성되지 않아 따로 지정해서 빌드했다.
```
petalinux-package --boot --fpga <FPGA bitstream> --u-boot
```
FIT에 대한 설명은 [[3]](https://elinux.org/images/f/f4/Elc2013_Fernandes.pdf)에 잘 나와있다.   

![FIT 파일 구조](/assets/image/fit_diagram.PNG)  

PETALINUX는 기본적으로 Initramfs mode가 활성화 되어있고 다음 dependancy를 만족한다.  

*  kernel needs rootfs to be built for initramfs
*  Building rootfs builds FSBL, PMUFW and ATF
*  Device-tree needs kernel headers
*  U-Boot needs device-tree, as it compiles with the External Device Tree 

[[4]Xilinx Wiki](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842374/U-Boot+Images)에 나와있는 FIT 이미지의 예시 layout은 다음과 같다.
```
/dts-v1/;
  
/ {
    description = "U-Boot fitImage for plnx_aarch64 kernel";
    #address-cells = <1>;
  
    images {
        kernel@0 {
            description = "Linux Kernel";
            data = /incbin/("./Image");
            type = "kernel";
            arch = "arm64";
            os = "linux";
            compression = "none";
            load = <0x80000>;
            entry = <0x80000>;
            hash@1 {
                algo = "sha1";
            };
        };
        fdt@0 {
            description = "Flattened Device Tree blob";
            data = /incbin/("./system.dtb");
            type = "flat_dt";
            arch = "arm64";
            compression = "none";
            hash@1 {
                algo = "sha1";
            };
        };
        ramdisk@0 {
            description = "ramdisk";
            data = /incbin/("./ramdisk.cpio");
            type = "ramdisk";
            arch = "arm64";
            os = "linux";
            compression = "none";
            hash@1 {
                algo = "sha1";
            };
        };
    };
    configurations {
        default = "conf@1";
        conf@1 {
            description = "Boot Linux kernel with FDT blob + ramdisk";
            kernel = "kernel@0";
            fdt = "fdt@0";
            ramdisk = "ramdisk@0";
            hash@1 {
                algo = "sha1";
            };
        };
    };
};
```

Image.ub에 대한 정보를 찾기 어려웠는데 [2]에 다음이 나와있다.
> Note: Xilinx tends to use .ub extension (i.e. Yocto generates the fitimage.itb or fit.itb file in deploy/images/<machine_name> directory and PetaLinux tools will rename this to image.ub while copying to <plnx-proj-root>/images/linux directory.)

sd 카드에 BOOT.BIN, Image.ub를 넣어 부팅에 성공하였다.  
`uname -ar`   
`Linux hellomodule 4.19.0-xilinx-v2019.1 #1 SMP PREEMPT Sun Sep 1 12:59:27 UTC 2019 armv7l GNU/Linux`


components/plnx_workspace/device-tree/device-tree/pl.dtsi를 보면 생성한 led IP에 대한 dt를 확인할 수 있다.
```
/ {
    amba_pl: amba_pl {
				#address-cells = <1>;
				#size-cells = <1>; 
				compatible = "simple-bus";
				ranges ;
        helloled_0: helloled@43c00000 {
								clock-names = "s00_axi_aclk";
								clocks = <&clkc 15>;
								compatible = "xlnx,helloled-1.0";
								reg = <0x43c00000 0x10000>;
								xlnx,s00-axi-addr-width = <0x4>;
								xlnx,s00-axi-data-width = <0x20>;
								};
			 };
};
```
components/plnx_workspace/device-tree/device-tree/drivers 디렉토리에 드라이버를 써 넣는것 같다. 하지만 yocto module으로 드라이버를 개발하도록 하겠다.  
[[2]](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1144-petalinux-tools-reference-guide.pdf)의 **Creating and Adding Custom Modules**(63p)을 참고하자.  

```
 petalinux-create -t modules --name ledmodule --enable  
```
	 
meta-user/recipes-module/ledmodule/files/ledmodule.c 생성되었다.  
기본 템플릿은 platform driver이다.
	
```
146         { .compatible = "xlnx,helloled-1.0", },
```