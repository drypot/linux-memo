# Arch Linux Install, Fdisk

<https://www.archlinux.org>\
<https://wiki.archlinux.org/index.php/Installation_guide>

<https://whjeon.com/arch-install/>

First: 2014.01.21, 서버에 설치한 후 메모를 정리.\
Update: 2021-02-06, 데스크탑에 설치한 후 내용 추가.

## 동기

2014년, Ubuntu 서버를 쓰고 있었다.\
패키지 디펜던시가 꼬여서 새버전 패키지를 설치할 수 없었다.\
다 밀어버리고 Arch를 설치했다.

2021년, 새로 구입한 데탑PC에 우분투 데스크탑을 설치하려고 했다.\
보드는 신형이고 우분투 커널은 1년 전 것이라 네트웍을 잡지 못했다.\
Arch를 설치했다.

이렇게 정리해도 다음에 설치할 때는 다시 원문을 봐야 한다.\
바뀐 것이 많을 것이다.

## Arch Linux

아치 리눅스는 우분투나 레드헷과 다르게 연도별 배포판이 없다.\
업데이트하면 최신 버젼이 된다.

패키지 매니저는 있지만 인스톨러가 없다.\
설치는 쉘과 텍스트 에디터로 수작업한다.

nano 나 vim 에디터를 쓸 줄 알아야한다.


## Download

<https://www.archlinux.org/download/>

USB 스틱을 만들어 부팅한다.\
새 시스템을 만들려면 인터넷에 접속된 상태여야한다.

## Connect to the internet

### DHCP

dhcp 클라이언트가 기본으로 동작한다.\
공유기 안이라면 인터넷에 이미 연결되어 있을 것이다.

확인.

    # ping www.google.com

### Static IP

수동 설정이 필요한 경우만 아래 작업을 한다.

dhcp 클라이언트를 정지.

    # systemctl stop dhcpcd

이더넷 인터페이스 확인.

    # ip link

    1: lo: <LOOPBACK,UP,LOWER_UP>
    2: eth0: <BROADCAST,MULTICAST>
    3: wlp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP>

인터페이스 활성화.

    # ip link set eth0 up

IP 부여.

    # ip addr add 192.168.1.2/24 dev eth0

IP 확인.

    # ip addr

게이트웨이 지정

    # ip route add default via 192.168.1.1

네임서버, 로컬 도메인 이름 설정

    # vi /etc/resolv.conf

    nameserver 168.126.63.1
    nameserver 168.126.63.2

여기까지 세팅했으면 네트웍이 작동해야한다.

    # ping www.google.com

참고, 구글 DNS

    nameserver 8.8.8.8
    nameserver 8.8.4.4

참고, 부여되어 있는 어드레스 모두 삭제.

    # ip addr flush dev eth0


### WiFi

<https://wiki.archlinux.org/index.php/Network_configuration/Wireless>

### Proxy

프락시 안에 있을 경우.

<https://wiki.archlinux.org/index.php/Proxy_settings>

## Update the System Clock

NTP 서버에서 현재 시각을 가져와 시스템 클럭을 세팅한다.

    # timedatectl set-ntp true

리눅스 운영체제 시계를 시스템 클럭이라고 한다.\
메인보드의 시계는 하드웨어 클럭 또는 리얼 타임 클럭이라고 한다.

## Partition 정책 결정

<https://wiki.archlinux.org/index.php/Installation_guide#Partition_the_disks>

디스크 파티션 방식을 결정한다.

BIOS 메인보드면 MBR이 무난하다.\
UEFI이면 GPT가 무난하다.

UEFI로 부팅됐으면 아래 폴더에 뭐가 잔뜩 들어있다.

    # ls /sys/firmware/efi/efivars

### MBR

프라이머리 파티션은 4 개 까지.\
다룰 수 있는 디스크 최대 크기는 2 TB.

디스크 맨 앞에 512 byte MBR 을 넣는다.

첫 파티선은 보통 512 byte * 2048 sectors = 1 MB 위치에 만든다.

MBR과 첫 파티션 사이를 Post MBR Gap이라 한다.\
GRUB core.img 가 여기에 들어간다.

요즘 fdisk 는 첫 파티션 위치를 2048 섹터로 권장한다.\
과거의 4 KiB 섹터 문제는 사라졌다


### GPT

파티션 갯수 무제한.

부팅용 디스크에는 EFI System Partition이 필요하다.\
ESP는 부트로더나 EFI 어플리케이션을 심는 OS 독립적인 파티션이다.\
크기는 256MB 나 512MB 정도 잡는다.\
파티션 생성후 파티션 타입을 EFI ... 로 지정한다.\
추후 FAT32로 포멧한다.

fdisk로 파티션을 생성하면 타입을 Linux filesystem으로 세팅한다.\
추후 고민없이 ext4 파일 시스템을 사용할 것이다.\
대부분의 경우 무난하다.

Swap 파티션을 생성하면 타입을 Linux swap으로 세팅한다.\
스웝 파티션은 systemd에서 인식해서 스웝으로 사용한다.\
하지만 우리는 fstab 정의에 명시적으로 표시할 것이다.

### Swap partition

<https://wiki.archlinux.org/index.php/Swap>

클라이언트용 시스템에서 Hiberate할 때 사용된다.\
스웝이 램 크기보다 작아도 Hibernate할 수 있다.\
스웝을 크게 잡고 싶어도 램 크기의 두 배 이상은 낭비다.

### Btrfs Partitioning

MBR, GPT 대신 Btrfs 를 스토리지 전체에 쓸 수 있다.\
Swap 파일을 Btrfs 파티션에 만들 수 없다.

### Windows

* BIOS + MBR : 부팅 가능
* BIOS + GPT : 부팅 불가
* UEFI + MBR : 부팅 불가
* UEFI + GPT : 부팅 가능

## Partition 생성

파티션 잡을 블럭 디바이스 확인.

    # lsblk

    # fdisk -l

fdisk로 MBR 또는 GPT 파티션 테이블을 구성한다.

    # fdisk /dev/<device>

2014년엔 gdisk가 필요했었다.\
2021년엔 fdisk로 모두 할 수 있었다.\
parted는 매우 불편하다.\
cfdisk라고 UI가 화려한 것이 있다.

파티션 작업 예 1, MBR.

    Device     Boot Start       End   Sectors   Size Id Type
    /dev/sda1  *     2048 488397167 488395120 232.9G 83 Linux

파티션 작업 예 2, GPT.

    Device         Start       End   Sectors   Size Type
    /dev/sdb1       2048   1050623   1048576   512M EFI System
    /dev/sdb2    1050624  68159487  67108864    32G Linux filesystem
    /dev/sdb3   68159488 135268351  67108864    32G Linux swap
    /dev/sdb4  135268352 202377215  67108864    32G Linux filesystem
    /dev/sdb5  202377216 488397134 286019919 136.4G Linux filesystem

## Format

블럭 디바이스 목록 확인.

    # lsblk

EFI System 파티션은 FAT32로 포멧한다.

    # mkfs.fat -F32 /dev/sd??

일반 파티션은 ext4로 포멧한다.

    # mkfs.ext4 /dev/sd??

-m 0 : 시스템 파티션외 데이터 파티션 포멧시에는 이 옵션으로 5% 의 디스크 낭비를 막을 수 있다.

## Swap

스웝 파티션을 포멧한다.

    # mkswap /dev/sd??

스웝을 켠다.

    # swapon /dev/sd??

확인.

    # swapon --show

## Mount The Partitions

루트 파일시스템을 /mnt에 마운트한다.

    # mount /dev/sda2 /mnt

마운트할 파일시스템들이 더 있다면 mnt 아래에 쭉 마운트한다.

기타 파티션 마운트 예. 

    # mkdir /mnt/boot
    # mkdir /mnt/var
    # mkdir /mnt/home

    # mount /dev/sda1 /mnt/boot
    # mount /dev/sda4 /mnt/var
    # mount /dev/sda5 /mnt/home
