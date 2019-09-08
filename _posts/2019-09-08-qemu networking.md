---
title: qemu networking
layout: posts
author_profile: true
read_time: true
comments: null
share: true
related: true
Categories: Programming
---

gdb 사용을 하며 계속 문제가 생겨 우선 네트워크 설정부터 만져보기로 했다.

~~어쩌다보니 qemu 설정에서 계속 막히게 되었다. 하지만 디바이스 드라이버를 개발하기 위해선 꼭 필요하므로 계속 삽질을 이어나갔다.~~

~~qemu의 네트워크 설정을 위해 브리지를 생성하고 tap을 생성하자. [[1]](https://gist.github.com/extremecoders-re/e8fd8a67a515fee0c873dcafc81d811c)~~

~~그런데 petalinux-boot의 기본 네트워크 설정이 obsolete된 -net 옵션이다. 따라서 tap 설정에 관한 참고자료가 많이 없는데  [[1]](https://gist.github.com/extremecoders-re/e8fd8a67a515fee0c873dcafc81d811c)에 나온대로 해보고 안되면 콘솔에 qemu-system-aarch64를 직접 타이핑 하여 해결해야 하는데 이마저도 에러가 난다.~~

~~` -M 'arm-generic-fdt-7seriese' machine not found`~~

~~총체적 난국이다...~~

드디어 방법을 찾았다. 우선 --qemu-args에 -net 옵션을 집어넣으면 petalinux-boot에서 자동으로 생성되는 -net 관련 옵션들은 생성되지 않는다. 따라서 앞에 서술한 대로 qemu-system-aarch64를 직접 타이핑 하지 않아도 임의로 네트워크 설정을 조작할 수 있다. 

qemu의 네트워킹에 대해 한번 정리하는게 좋을 것 같아서 이번 포스트에서 정리하도록 하겠다. -net 옵션에 대한 설명이 공식 위키에 없어서 삽질을 좀 했다. 처음에는 ICMP가 막혀있고 10.0.2.2에서 incomig packet이 모두 막혀있으므로 이전에 내가 네트워크 설정을 확인하며 했던 ssh와 ping 명령어가 모두 먹히지 않았다.

다음을 보자 [[1]](https://wiki.qemu.org/Documentation/Networking) [[2]](https://stackoverflow.com/questions/22654634/difference-between-net-user-and-net-nic-in-qemu) petalinux-boot에 기본으로 제공되는 arg string `-net nic -net user`은 [[2]](https://stackoverflow.com/questions/22654634/difference-between-net-user-and-net-nic-in-qemu)에 따르면 `-net nic`가 guest의 네트워크 인터페이스를 생성하고 `-net user`가 [[1]](https://wiki.qemu.org/Documentation/Networking)의 slirp 생성하여 이 둘을 연결한다. 따라서 이 상황에서도 외부 네트워크에 접근이 가능하다.  다음 포스트를 참고하면 tap/port forwarding  설정이 가능하다 [[3]](https://www.joinc.co.kr/w/Site/cloud/Qemu/Network).

ping을 쓰기 위해서는 호스트의 tap 설정과 qemu arg `-net nic -net tap,ifname=tap0,script=no`의 설정을 해주면 되고 ssh만 쓰기 위해서는 port forwarding 설정 `-net nic -net user,hostfwd=tcp::60022-:22`을 해주면 된다.

결과적으로는 네트워크 옵션은 잘 설정되 있었다. 다만 확인하는 방법이 잘못됬을 뿐이다. `-gdb`옵션을 앞으로 확인해 보자.