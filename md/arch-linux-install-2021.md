# Arch Linux Install 2021 01


## History

2014-01-21: 서버 설치 후 메모 정리.\
2021-02-06: 데스크탑 설치 후 내용 추가.


## Arch Linux

<https://www.archlinux.org>\
<https://wiki.archlinux.org/index.php/Installation_guide>\
<https://whjeon.com/arch-install/>

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
    /dev/sdb3   68159488 135268351  67108864    32G Linux filesystem
    /dev/sdb4  135268352 454842367 319574016 152.4G Linux filesystem
    /dev/sdb5  454842368 488396799  33554432    16G Linux swap


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
    # mount /dev/sda3 /mnt/var
    # mount /dev/sda4 /mnt/home


## Mirrors

미러 리스트를 업데이트한다.

    # reflector

reflector 는 접속 속도에 따라 적절한 /etc/pacman.d/mirrorlist 를 생성한다.\
이 파일은 새 시스템에도 복사된다.


## Install the Base System

아치는 패키지를 무더기로 넣지 않는다.\
사용자가 설치할 패키지를 모두 선택해야 한다.

꼭 설치해야할 패키지들.

    base linux linux-firmware

초반에 거의 설치해야 할 패키지들.

    vim nano  # 에디터 선택
    base-devel
    networkmanager
    man-db man-pages

네트웍관리 패키지들이 많다.\
networkmanager 를 선택한 것은 gnome 에서 어차피 저 놈을 설치하기 때문이다.

기타 개인적 필요에 의한 패키지들.

    git zsh fzf

레이드등 디스크를 복잡하게 구성할 때 필요한 패키지들.

    mdadm lvm2 

패키지들을 골랐으면 설치한다.

    # pacstrap /mnt base linux linux-firmware vim base-devel networkmanager man-db man-pages

pacstrap 에서 까먹은 패키지들은 아래 chroot 한 상태에서 pacman 으로 하나하나 설치 가능하다.


## fstab

fstab 생성.

    # genfstab -U /mnt >> /mnt/etc/fstab

-U: 부팅할 때 장치 순서가 바뀔 수 있으니 되도록 UUID 를 쓴다.

fstab 의 마지막 필드는 파일 시스템 체크 순서.

    1: root
    2: root 이외 파티션은 2 로 한다.
    0: do not check. Btrfs 는 0 이어야 한다.

fstab 항목들은 부팅시 systemd mount units 들로 자동 변환된다.


## chroot

루트 변경

    # arch-chroot /mnt


## Time zone

타임 존을 설정.

    # ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime

리눅스는 셧다운할 때마다 메인보드 클럭에 시스템 클럭을 덮어쓴다.\
이 값을 UTC로 할지 Localtime으로 할지 정해야 한다.\
리눅스 기본은 UTC를 쓰는 것이다.

    # hwclock --systohc

결과를 확인.

    # cat /etc/adjtime

윈도우는 메인보드 클럭에 로컬 타임을 넣는다.\
윈도우와 듀얼부팅하는 경우 리눅스도 로컬 타임을 넣도록 할 필요가 있다.

    # timedatectl set-local-rtc 1

`hwclock -w -l` 명령으로도 같은 기능을 할 수 있을 것 같았는데 잘 안 됐다.

<https://wiki.archlinux.org/index.php/System_time#Time_standard>


## Locale

생성할 locale 을 선택한다.

    /etc/locale.gen

    en_US.UTF-8 UTF-8 을 언코멘트

locale 생성.

    # locale-gen

locale.conf 생성

    # echo LANG=en_US.UTF-8 > /etc/locale.conf
    export LANG=en_US.UTF-8


## Console Font and Keymap

...


## Hostname

호스트이름 지정, 예.

    # echo drypot-arch > /etc/hostname

hosts 파일 생성.

    /etc/hosts

    127.0.0.1   localhost
    ::1         localhost
    127.0.1.1   drypot-arch

고정IP를 사용하면 127.0.1.1 대신 입력해 둔다.


## Network Configuration

<https://wiki.archlinux.org/index.php/Network_configuration>

2014년엔 netctl 패키지를 설치하고 고정 IP 설정을 해서 꽤 복잡했습니다.\
너무 오래된 내용이라 일단 옮기지 않았습니다.

2021년엔 networkmanager 패키지를 설치하고 dhcp를 써서 거의 설정이 필요 없었습니다.

부팅할 때마다 NetworkManager가 동작하도록 활성화합니다.

    # systemctl enable NetworkManager.service

전 이걸 아래 다 완료하고 재부팅한 후에 했습니다.\
설치만으로 enable 되지 않아서 네트웍이 안되더군요.\
혹시 이 단계에서 에러나면 재부팅 후 하셔도 됩니다.


## Initramfs

혹시 LVM, RAID 패키지를 추가 설치했으면 initramfs 를 다시 만들어야 한다.\
보통은 필요없다.

    # mkinitcpio -P


## Root Password

루트 패스워드 지정.

    # passwd


## Boot Loader


### BIOS + MBR

<https://wiki.archlinux.org/index.php/GRUB>

아래 GRUB 내용은 2014년도 판이다.\
이후 변경된 내용은 확인하지 못했다.

패키지 설치.

    # pacman -S grub

설정 생성.

    # grub-mkconfig -o /boot/grub/grub.cfg

grub.cfg 에 삽입된 UUID 가 /etc/fstab 의 루트와 맞는지 재확인한다.

MBR 에 로더를 설치한다.

    # grub-install --target=i386-pc --recheck --debug /dev/sda

    --target=i386-pc : BIOS 타입이란 것을 명시적으로 지정

미러를 사용할 경우 해당 디스크 모두에 로더를 설치한다.


### UEFI + GPT

<https://wiki.archlinux.org/index.php/Boot_loader>

2021년 2월 버전이다.

아치는 UEFI로 부팅되어 있어야 한다.

이전 과정을 통해 EFI System Partition 이 /boot에 마운트되어 있어야 한다.

GPT용 부트로더가 많은데 간단하다고 해서 systemd-boot 를 사용했다.\
systemd 패키지에 딸려오므로 추가 패키지 설치가 필요없다.

systemd-boot를 EFI System Partition에 설치한다.

    # bootctl install

아래 두 파일을 만들어 준다.

부트 로더 전체 설정.

    /boot/loader/loader.conf

    default arch
    editor 1
    timeout 3

arch 항목에 대한 디테일.

    /boot/loader/entries/arch.conf

    title Arch Linux
    linux /vmlinuz-linux
    initrd /initramfs-linux.img
    options root=PARTUUID=XXXXXXXXXXXXXXXXXXXXXXX rw

마지막 줄에 장치 UUID 를 읽어다 써야 합니다.\
vi 에서 아래처럼 하면 실행 결과가 vi 안으로 들어옵니다.

    :r! blkid -s PARTUUID -o value /dev/sdXn


## Unmount and Reboot

아직 꼽혀있다면 설치용 USB스틱을 제거합니다.

chroot 환경 종료.

    # exit

언마운트.

    # umount -R /mnt

리부트.

    # reboot


## Root Login

재부팅이 완료되면 root로 로그인한다.


## Check date

날짜가 정상인지 확인한다.

    # date


## User management

사용자 추가

    # useradd -m -g wheel -G [additional_groups] <username>

    -m: create home directory
    -g wheel: 관리자 그룹에 등록

예.

    # useradd -m -g wheel drypot

비밀번호 설정.

    # passwd drypot


## Sudo

설정파일은 아래와 같다.

    /etc/sudoers

문법 오류나면 sudo 안 되면서 망할 수 있으므로 가능하면 visudo 로 편집한다.

    $ sudo sh
    # EDITOR=vim visudo

파일 상단 적당한 곳에 아래 내용을 넣는다.

    %wheel ALL=(ALL) ALL
    Defaults timestamp_timeout=60
    Defaults !tty_tickets

wheel 그룹 모두가 sudo를 쓸 수 있게 한다.\
패스워드 재입력 시간을 60 분으로 설정한다.\
터미널 별로 패스워드 입력받는 것을 막는다.


## ufw

2014년도 내용이라 확인이 필요하다.

iptables 가 비활성화 (Active: inactive) 상태인 것을 확인한다. ufw 와 쫑난다.

    # systemctl status iptables
    # systemctl status ip6tables

설치.

    # pacman -S ufw

    # systemctl start ufw
    # systemctl enable ufw

설정.

    # ufw enable

    # ufw allow ssh 
    # ufw allow http <-- udp 로 http 보내는 스팩이 있으니 tcp 로 제한하진 말자;
    # ufw allow https

    # ufw status

smtp 는 열지 않아도 메일 발송에 문제가 없다.


## lm_sensors

2014년도 내용이라 확인이 필요하다.

온도 모니터링 모듈 설치.

    # pacman -S lm_sensors

설정. 모르면 Enter 친다.

    # sensors-detect

온도 덤프.

    # sensors

