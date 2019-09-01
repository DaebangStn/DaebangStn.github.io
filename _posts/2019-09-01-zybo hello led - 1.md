---
title: zybo hello led - 1
layout: posts
author_profile: true
read_time: true
comments: null
share: true
related: true
categories:
- Programming
---

FPGA 연산 가속을 위한 step 프로젝트로 zybo에서 LED blink IP를 만들고 petalinux tool으로 빌드하여 디바이스 드라이버까지 작성해보는 작업을 할 것이다.

**reference 분석**  
[[1] instructable](https://www.instructables.com/id/Embedded-Linux-Tutorial-Zybo/) gcc로 직접 u-boot, 커널, 모듈 빌드하여 작업함  
[[2] petalinux reference guide](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1144-petalinux-tools-reference-guide.pdf)  


우선 zybo board file을 다운로드 하고, [1]과 같이 IP를 생성하여 export Hardware을 마무리 하였다.  
[2]의 47p까지 진행  
우선 ~/petalinux/2019.1/setting.sh source  
petalinux project 생성  
```
petalinux-create -t project -n name -t zynq
```
petalinux config (ssh에서는 깨짐)
```
petalinux-config --get-hw-description=<path to .hdf directory, vivado project .sdk directory>
```
petalinux build  
```
petalinux-build
```