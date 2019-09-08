---
title: introduction to platform driver - 1
layout: posts
author_profile: true
read_time: true
comments: null
share: true
related: true
categories: Programming
---

이전 zybo hello led 포스트에서 zybo 보드를 위해 PS, PL block를 설정하고 petalinux tools를 이용해 부팅에 성공했다.  이제 디바이스 드라이버를 작성해서 LED가 점멸하는것을 확인해야 한다.  디바이스 드라이버 작성법을 앞으로 알아보자. 진행하기에 앞서 Bootlin의 좋은 자료 [[1]](https://bootlin.com/doc/training/linux-kernel/linux-kernel-slides.pdf)를 한번 읽어보자.  

본격적인 개발에 들어가기에 앞서 qemu 설정을 미리 해주고 가자.  [[2]](https://www.xilinx.com/support/documentation/sw_manuals/petalinux2013_10/ug982-petalinux-system-simulation.pdf)

>apt install qemu && petalinux-boot --qemu --kernel

앞서 ledmodule의 compatible 설정을 바꾸어 주었으면 부팅시 다음과 같은 로그를 볼 수 있다.


```
ledmodule 43c00000.helloled: Device Tree Probing
ledmodule 43c00000.helloled: no IRQ found
ledmodule 43c00000.helloled: ledmodule at 0x43c00000 mapped to 0xe0960000
```

다음 명령을 통해 gdb 커널 디버깅이 가능하다.

```
$ cd "<project-root>/images/linux"
$ petalinux-util --gdb vmlinux
(gdb) target remote :9000
(gdb) continue
```

하지만 

```
(gdb) c
The program is not being run.
```

메세지와 함께 qemu가 freeze 되는 현상이 발생한다. 이는 커널 빌드시 디버깅 옵션이 설정되지 않아 발생하는 것 같다. [[3]](https://www.xilinx.com/support/answers/66853.html), [[4]](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/123011146/Linux+Debug+infrastructure+Kernel+debugging+using+KGDB)

우선 kgdb를 enable 해 보자.

>petalinux-config -c kernel

Kconfig에서 관련 설정들을 건드릴 수 있다.  하지만 여전히 qemu는 gdb  target 설정시 freeze된다. 아무래도 부팅로그

>qemu vlan 0 is not connected to host network

[[4]](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_1/ug1169-xilinx-qemu.pdf)을 참고하여 qemu의 네트워크 설정을 진행하였다.

2019.09.08
위에 나온 vlan0 로그 메세지는 petalinux-boot에서 default로 사용하는 vlan의 vid가 1이라 발생하는 일이다. 하~나~도 문제될것 없다.