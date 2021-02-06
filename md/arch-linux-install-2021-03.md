# Arch Linux Install 2021 03

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


## sshd

2014년도 내용이라 확인이 필요하다.

설치.

    # pacman -S openssh

데몬을 돌리려면.

    # systemctl start sshd
    # systemctl enable sshd

설정.

    /etc/ssh/sshd_config

    AllowUsers user1 user2 <-- 특정 유저만 접속 가능하게
    PermitRootLogin no <-- 루트 로그인 불가능하게


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

