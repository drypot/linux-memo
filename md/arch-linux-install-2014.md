# Installing Arch 2014

Last Update: 2014.01.21

오래 업데이트되지 않았습니다.\
참고로만 보세요.

<https://www.archlinux.org>\
<https://wiki.archlinux.org/index.php/Installation_guide>

## Arch Linux

아치 리눅스는 우분투나 센트와 다르게 연도별 배포판이 없다.
업데이트하면 최신버젼이 된다.

패키지 매니저는 있지만 인스톨러가 없다.
설치는 쉘과 텍스트 에디터로 수작업 진행한다.

아래 내용은 vi 와 bash 을 쓸 줄 안다는 가정하에 썼다.


## Download

아치 이미지는 32, 64 비트를 동시에 지원한다.
새 시스템을 설치하려면 반드시 인터넷에 접속된 상태여야한다.

<https://www.archlinux.org/download/>


## Partitioning

### Swap partition

기가단위로 램이 있는 시스템은 스웝이 불필요하다.
클라이언트용 시스템에서 suspend-to-disk 쓸 경우는 필요하다.

<https://wiki.archlinux.org/index.php/Swap>


### BIOS + MBR

시스템 하드는 GPT 대신 MBR 쓰는 것이 편한 것 같다.

MBR 의 프라이머리 파티션은 4 개 까지.
MBR 이 다룰 수 있는 최대 크기는 2 TB.

디스크 맨 앞에는 512 byte MBR 이 있다.

첫 파티선은 보통 512 byte * 2048 sectors = 1 MiB 위치에 만든다.

MBR 과 첫 파티션 사이를 Post MBR gap 이라 하는데 GRUB core.img 가 여기에 들어간다.

요즘 fdisk 는 첫 파티션 위치를 2048 섹터로 권장한다.
과거의 4 KiB 섹터 문제는 사라졌다

블럭 디바이스 확인.

    # lsblk <-- -f 붙이면 uuid 까지 나온다.

MBR 은 fdisk 로 생성한다.

    # fdisk /dev/<device>

    p: print partition table.
    o: create new MBR partition table.
    n: add a new partition.
    v: verify the partition table.
    w: save and exit


### BIOS + GPT

왠만하면 쓰지 않는다.

그래도 쓰겠다면 Post MBR gap 이 없기 때문에 GRUB core.img 를 심을 별도의 부트 파티션이 필요하다.
부트 파티션 위치는 2 TiB 안이면 되지만 가능하면 디스크 앞쪽에 1007 KiB 크기로 만든다.
1007 KiB + 맨앞 17 KiB GPT 하면 딱 1024 KiB 이 된다.

gdisk 에서 지정할 부트 파티션 타입은 ef02 이다.


### UEFI + MBR

하지 말라;


### UEFI + GPT

GPT 는 파티션 갯수 무제한.

EFI 부팅용 디스크에는 EFI System Partition 이 필요하다.
EFS 는 부트로더나 EFI 어플리케이션을 심는 OS 독립적인 파티션이다.

GPT 편집에는 gdisk 를 사용한다.
gdisk 는 파티션 바운더리를 자동으로 1024 KiB 에 맞추므로 대부분의 SSD 와 호환성을 가진다.

    # gdisk /dev/sda

    p: print
    o: new partition table
    n: new partition
    v: verify before save
    w: save and exit
    q: quit without saving

EFS 타입은 ef00 이다.
Arch 에서 권장하는 EFS 크기는 512 MiB 이다.
윈도우는 100 MiB 쓴다고 한다.

EFS 파티션 생성후엔 FAT으로 포멧한다.

    # mkfs.fat -F32 /dev/<partition>


### Btrfs Partitioning

MBR, GPT 대신 Btrfs 를 스토리지 전체에 쓸 수 있다.

Swap 파일을 Btrfs 파티션에 만들 수 없다.
Swap 을 쓰려면 별도 파티션을 만든다.


### Windows

* BIOS + MBR : 부팅 가능
* BIOS + GPT : 부팅 불가
* UEFI + MBR : ...
* UEFI + GPT : 부팅 가능

### 기타

<https://wiki.archlinux.org/index.php/Partitioning>


## Format

블럭 디바이스 목록 확인.

    # lsblk

포멧.

    # mkfs.ext4 /dev/<partition>

    -m 0 : 시스템 파티션외 데이터 파티션 포멧시에는 이 옵션으로 5% 의 디스크 낭비를 막는다.


## Mount the partitions

루트 파일시스템 마운트.

    # mount /dev/sda1 /mnt

기타 마운트할 파티션 있으면 마운트.

    # mkdir /mnt/home
    # mount /dev/sda2 /mnt/home


## Connect to the internet

DHCP 서비스가 디폴트로 동작한다.
DHCP 를 쓰지 않을 것이라면 정지.

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
    search example.com

여기까지 세팅했으면 네트웍이 작동해야한다.

    # ping www.google.com

참고, 구글 DNS

    nameserver 8.8.8.8
    nameserver 8.8.4.4

참고, 부여되어 있는 어드레스 모두 삭제.

    # ip addr flush dev eth0


### WiFi

무선 장치 확인. 보통 'w' 로 시작.

    # iw dev

    phy#0
    Interface wlp3s0

장치가 안 뜬다면 드라이버가 로딩되지 않은 상태. 아래 참고.\\
<https://wiki.archlinux.org/index.php/Wireless_Setup>

인터페이스 활성화

    # ip link set wlp3s0 up

와이파이 설정

    # wifi-menu wlp3s0

여기까지 세팅했으면 네트웍이 작동해야한다.


### Proxy

프락시 안에 있을 경우.\\
<https://wiki.archlinux.org/index.php/Proxy_settings>


## Mirrors

미러리스트는 pacstrap 에 의해 새 시스템에 복사되니 여기서 미리 잘 편집해 둔다.
keepalive 때문에 HTTP 가 FTP 보다 훨 빠르다.

미러 리스트에서 kaist 를 찾아 목록 제일 위로 올린다.

    /etc/pacman.d/mirrorlist

미러 리스트를 수정했으면 패키지 목록을 반드시 업데이트한다.

    # pacman -Syy

    -yy: -y 를 쌍으로 쓰면 더 강력한 업데이트를 한다.

<https://wiki.archlinux.org/index.php/mirrors>


## System date time

시스템 클럭 + 소프트웨어 클럭을 업데이트.

    # ntpd -qg

    -q : 데몬으로 남지 않고 실행 후 바로 종료.
    -g : NTP 시간과 시스템 클럭간 차가 많더라도 강제 사용한다.

## Install the base system

베이스 시스템 설치.

    # pacstrap /mnt base


## fstab

fstab 생성.

    # genfstab -p -U /mnt >> /mnt/etc/fstab

    -U: 부팅할 때 장치 순서가 바뀔 수 있으니 되도록 UUID 를 쓴다.

fstab 의 마지막 필드는 파일 시스템 체크 순서.

    1: root
    2: root 이외 파티션은 2 로 한다.
    0: do not check. Btrfs 는 0 이어야 한다.

fstab 항목들은 부팅시 systemd mount units 들로 자동 변환된다.


## chroot

루트 변경

    # arch-chroot /mnt


## Locale

생성할 locale 을 선택.

    /etc/locale.gen

    en_US.UTF-8 UTF-8 을 언코멘트

locale 생성.

    # locale-gen

locale.conf 생성

    # echo LANG=en_US.UTF-8 > /etc/locale.conf
    # export LANG=en_US.UTF-8


## Console font and keymap

...


## Time zone

    # ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime


## Hardware clock

하드웨어 클럭에 UTC 타임을 넣을 것인지 로컬 타임을 넣을 것인지 결정해야한다.

리눅스에서 권장하는 방법은 UTC 를 넣는 것이다.

    # hwclock -w --utc

이 명령으로 /etc/adjtime 파일이 자동 생성된다.

리눅스에서 로컬 타임을 쓰면 다양한 문제가 발생한다.
윈도우는 로컬 타임을 쓴다.
고로 윈도우와 리눅스를 듀얼 부팅하면 다양한 문제가 발생한다.


## Service management

아치는 서비스 관리에 systemd 를 사용한다.

부팅시 실행되도록 서비스를 등록한다. 바로 실행되진 않는다.

    # systemctl enable <service>

서비스를 바로 실행한다.

    # systemctl start <service>

상태 확인.

    # systemctl status <service>

<https://wiki.archlinux.org/index.php/Systemd>


## NTP

설치.

    # pacman -S ntp
    # systemctl enable ntpd

설정.

    /etc/ntp.conf

    server 0.pool.ntp.org iburst
    server 1.pool.ntp.org iburst
    server 2.pool.ntp.org iburst

네트웍 상태가 좋지 않아 여러번 리퀘스트를 보냈다가 밴당하지 않기위해 iburst 옵션을 추가해 놓는다.


## hostname

호스트이름 지정.

    # echo myhostname.domain.com > /etc/hostname

Postfix 에서 도메인 이름을 가져다 사용하니 정확히 넣어두도록한다.


## Network for new root

### Dynamic IP / dhcpcd

유선 하나라면 network management service 필요없이 dhcpcd 만 활성화하면 된다.

    # systemctl enable dhcpcd


### Static IP / netctl

Arch 는 네트웍 관리를 위해 netctl 을 사용한다.

셈플 설정을 하나 복사해서 default 라는 사용자 프로파일을 만든다.

    # cd /etc/netctl
    # cp examples/ethernet-static default

systemd 가 '-' 을 특별히 인식하므로 프로파일 이름에 '-' 을 넣으면 안 된다.

인터페이스 이름을 확인한다.

    # ip link

프로파일을 편집한다. 예,

    Address=('10.211.55.100/24')
    Gateway='10.211.55.1'
    DNS=('168.126.63.1' '168.126.63.2')

이더넷 선의 연결 유무와 상관 없이 인터페이스를 활성화하려면 아래 줄을 추가한다.

    SkipNoCarrier=yes

재부팅시 프로파일이 활성화되도록 등록한다.

    # netctl enable default

"Running in chroot, ignoring request." 경고는 무시한다.

<https://wiki.archlinux.org/index.php/Netctl>


### Wireless

...


## Make ramdisk

램디스크 이미지는 mkinitcpio.conf 기본설정에 따라 이미 만들어져 있다.

참고로 mkinitcpio.conf 수정후 램디스크 재생성하려면.

    # vi /etc/mkinitcpio.conf
    # mkinitcpio -p linux


## Root password

루트 패스워드 지정.

    # passwd


## Boot loader

GRUB 은 GRUB version 2 를 의미한다.
구버전 grub-legacy 는 공식 지원이 중단되었다.

GRUB 은 Btrfs root 를 지원한다.

GRUB 이 지원하지 않는 파일시스템을 root 로 사용할 경우 /boot 용 파티션을 별도로 마련해야 한다.

<https://wiki.archlinux.org/index.php/GRUB>


### BIOS + MBR

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

BIOS 가 아니라 UEFI 로 부팅되어 있어야한다.

grub, dosfstools, efibootmgr 패키지를 설치한다.

    # pacman -S ...

EFI system partition 을 마운트한다.

    # mkdir /boot/efi
    # mount /dev/<esp> /boot/efi

GRUB EFI 어플리케이션을 설치한다.

    # grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --recheck --debug


## Unmount and reboot

chroot 환경 종료

    # exit

언마운트

    # umount -R /mnt

리부트

    # reboot


## Check network

재부팅 후 네트웍이 정상인지 확인한다.

    # netctl status default

오류가 있으면 프로파일을 수정하고 재기동해본다.

    # netctl start default

프로파일을 수정했으면 서비스를 재등록해야한다.

    # netctl reenable default

기타 오류 확인

    # journalctl -xn


## Check date

날짜가 정상인지 확인한다.

    # date


## Check ntpd

ntpd 작동하는지 확인한다.

    # systemctl status ntpd

    # ntpq -pn

    -p: print peers
    -n: numeric address


## bash

bash-completion 패키지 설치

    # pacman -S bash-completion


## User management

사용자 추가

    # useradd -m -g wheel -G [additional_groups] <username>

    -m: create home directory
    -g wheel: 관리자 그룹에 등록

비밀번호 변경.

    # passwd <username>


## sudo

sudo 는 root 비밀번호 대신 내 비밀번호를 입력하고 root 권한을 얻는 개념이다.

설치.

    # pacman -S sudo

설정파일은 /etc/sudoers 인데 문법 오류나면 sudo 안 되면서 망하므로 반드시 visudo 로 편집한다.

설정.

    # visudo

wheel 그룹이 sudo 쓸 수 있게 하려면 아래 설정을 찾아 언코멘트.

    %wheel ALL=(ALL) ALL

패스워드 재입력 시간을 60 분으로 설정하려면.

    Defaults timestamp_timeout=60

터미널 별로 패스워드 입력받는 것을 막으려면.

    Defaults !tty_tickets


## ssh

설치.

    # pacman -S openssh

    # systemctl start sshd
    # systemctl enable sshd

설정.

    /etc/ssh/sshd_config

    AllowUsers user1 user2 <-- 특정 유저만 접속 가능하게
    PermitRootLogin no <-- 루트 로그인 불가능하게


## ufw

파이어월 설치.

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

온도 모니터링 모듈 설치.

    # pacman -S lm_sensors

설정. 모르면 Enter 친다.

    # sensors-detect

온도 덤프.

    # sensors

 
## MDADM

MDADM 레이드 생성


## Data directory

    # mkdir /data
    # chown drypot:wheel /data
    # mount <device> /data

볼륨의 uuid 확인

    $ lsblk -f

    $ ls -l /dev/disk/by-uuid

fstab 에 uuid 추가

    /etc/fstab

    UUID=.... /data2 ext4 defaults 0 2
    UUID=.... /data3 ext4 defaults 0 2


## Bash

    ~/.bashrc

    alias lla='ls -alFv'
    alias ll='ls -lFv'
    alias la='ls -CFav'
    alias l='ls -CFv'

## Base-devel

C, make 등 기본 컴파일 도구 설치

    # pacman -S base-devel

## Git

    # pacman -S git


## Postfix

    # pacman -S postfix
    # systemctl enable postfix

설정.

    /etc/postfix/main.cf

    myhostname = ... <-- hostname 에서 자동으로 가져온다.
    mydomain = ... <-- hostname 에서 자동으로 가져온다.

    mydestination = $mydomain <-- 찾아서 언코멘트
    myorigin = $mydomain <-- 찾아서 언코멘트
    mynetworks_style = host <-- 찾아서 언코멘트

메일 발송 테스트. raysoda 프로젝트 루트에서.

    $ node server/mailer/mailer2-test-send.js

메일을 발송하려면 DNS SPF 레코드에 메일 서버 IP를 등록해놔야한다.

    v=spf1 include:aspmx.googlemail.com ip4:219.254.35.24 ip4:219.254.35.25 ip4:219.254.35.26 ip4:219.254.35.27 ~all

## Nginx

    # pacman -S nginx
    # systemctl enable nginx

기본 설정 파일 수정

    /etc/nginx/nginx.conf

    include /data/nginx/nginx.conf;

로컬 설정 파일 수정

    /data/nginx/nginx.conf

    ...

## certbot

Let's Encrypt 클라이언트 설치.

    # pacman -S certbot

acme 챌린지 디렉토리 생성

    # mkdir -p /var/lib/letsencrypt/.well-known
    # chgrp http /var/lib/letsencrypt
    # chmod g+s /var/lib/letsencrypt
    
certbot 이 동작할 수 있게 nginx default server 설정을 한다.

    server {
      listen 80 default_server;
      server_name _;
        
      location ^~ /.well-known {
        allow all;
        alias /var/lib/letsencrypt/.well-known/;
        default_type "text/plain";
        try_files $uri =404;
      }

      location / {
        return 301 https://$host$request_uri;
      }
    }

certbot 으로 인증서를 받아 심는다.

    certbot certonly --email drypot@gmail.com --webroot -w /var/lib/letsencrypt/ -d raysoda.com -d www.raysoda.com -d file.raysoda.com

nginx 사이트들의 ssl 설정을 마무리한다.

인증서 업데이트 자동화 1 단계.

    /etc/systemd/system/certbot.service

    [Unit]
    Description=Let's Encrypt renewal

    [Service]
    Type=oneshot
    ExecStart=/usr/bin/certbot renew --quiet --agree-tos
    ExecStartPost=/bin/systemctl reload nginx.service

인증서 업데이트 자동화 2 단계.

    /etc/systemd/system/certbot.timer

    [Unit]
    Description=Daily renewal of Let's Encrypt's certificates

    [Timer]
    OnCalendar=daily
    RandomizedDelaySec=1day
    Persistent=true

    [Install]
    WantedBy=timers.target

스타트.

    # systemctl enable certbot.timer
    # systemctl start certbot.timer

## MonogoDB

    # pacman -S mongodb
    # systemctl enable mongodb

    # mkdir /data/mongodb
    # chown mongodb:daemon /data/mongodb

설정.

    /etc/mongodb.conf

    dbpath=/data/mongodb

테스트.

    # systemctl start mongodb

    ...

mongo 에서 transparent_hugepage/enabled, transparent_hugepage/defrag 를 never 로 하라고 경고 뜨면, 
/etc/tmpfiles.d/local.conf 파일에 아래 내용 추가하고 재부팅

    w /sys/kernel/mm/transparent_hugepage/enabled - - - - madvise
    w /sys/kernel/mm/transparent_hugepage/defrag - - - - madvise
    w /sys/kernel/mm/transparent_hugepage/khugepaged/defrag - - - - 0


## Redis

    # pacman -S redis
    # systemctl enable redis

설정.

    /etc/redis.conf

    save 로 시작하는 부분 코멘트해서 디스크 저장을 못하게 한다.


## Node

    # pacman -S nodejs
    # pacman -S npm

    # pacman -S python2 <— npm gyp 등에서 사용.

    # npm config set prefix ~/npm <- sudo 없이 npm install -g 를 쓸 수 있다.

.bashrc 에 다음을 추가

    PATH=$PATH:~/npm/bin
    export PYTHON=python2

mocha, bower 등을 설치

    # npm install -g mocha

    ...

## Installing App Servers

해당 프로젝트의 INSTALL.md 참고.

