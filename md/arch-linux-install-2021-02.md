# Arch Linux Install 2021 02

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

기타 초반에 거의 설치해야 할 패키지들.

    vim nano  # 에디터 선택
    base-devel
    networkmanager
    man-db man-pages

네트웍관리 패키지들이 많다.\
networkmanager 를 선택한 것은 gnome 에서 어차피 저 놈을 설치하기 때문이다.

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

    127.0.0.1	localhost
    ::1	        localhost
    127.0.1.1	drypot-arch

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

