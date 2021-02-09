# uim Install 2021

초판: 2021-02-09

<https://wiki.archlinux.org/index.php/Localization_(한국어)/Korean_(한국어)>\
<https://wiki.archlinux.org/index.php/User:Isaac914/uim>

Arch Linux + Gnome + Wayland 에서\
uim + Byeoru 한글입력기를 설치하는 과정이다.

2021년 2월 기준. 

## IBus

설치가 쉽고 적당히 원만하게 작동한다.\
하루 사용하면서 두 가지 문제점이 있었다.\
트위터, 페이스북 웹 페이지에서 한글 입력이 안 된다.\
IntelliJ + IdeaVim 에서 한글을 입력하다가 ESC를 누르면 마지막 글자가 사라진다.

## uim

IBus를 참고 사용해도 됐었는데 uim으로 바꾸어 보았다.\
IBus의 두 가지 문제점은 해결되었다.

설치는 많이 까다로워서 이틀간 삽질했다.\
자동화된 패키지가 아직 없다.\
기존 문서들이 Xorg 기준으로 되어 있어서 Wayland에 맞지 않았다.\
한번에 쭉 따라할 수 있는 문서가 없었다.

## uim 패키지 설치

IBus 와 uim 을 나란히 두고 사용해도 될 것 같다.\
하지만 일단 IBus를 삭제하고 시작했다.

    $ sudo pacman -R ibus-hangul
    $ sudo pacman -R ibus

uim 패키지 설치.

    $ sudo pacman -S uim

Debian 문서들을 보면 apt-get으로 이정도 하면 그냥 된다고 나와있다.\
Arch 경우엔 아직 자동화가 안 된 것 같다.\
수작업을 해야 했다.

## Xorg 와 Wayland 차이

Xorg 세팅은 쉘 스크립트로 하면 됐었다.\
Wayland는 systemd를 사용하기 때문에 쉘 스크립트를 쓸 수 없다.

Arch 문서는 Xorg 기준으로 되어있다.\
Wayland에 설치하기 위해서는 수정해야 할 부분들이 있었다.

## uim 환경변수

처음엔 공식문서를 Wayland 기준으로 수정해서 아래처럼 했었다.

    ~/.config/systemd/user/uim-env.service

    [Unit]
    Description=uim environment initialization
    Before=graphical-session.target

    [Service]
    Type=oneshot
    ExecStart=/usr/bin/systemctl --user set-environment XMODIFIERS=@im=uim
    ExecStart=/usr/bin/systemctl --user set-environment GTK_IM_MODULE=uim
    ExecStart=/usr/bin/systemctl --user set-environment QT_IM_MODULE=uim

아치 문서의 xorg.target을 graphical-session.target으로 수정했다.

다 좋았는데 윈도우 시스템에 이벤트를 거니 환경변수 세팅 템포가 느렸다.\
자동 시작하는 프로그램들에 환경변수가 전달되지 않았다.

해서 아래처럼 environment.d 루틴에 환경변수 세팅을 맡겼다.

    ~/.config/environment.d/im.conf

    GTK_IM_MODULE=uim
    QT_IM_MODULE=uim
    XMODIFIERS=@im=uim

environment.d 도 systemd --user 세션 초기화 과정중 일부이다.\
일단은 이렇게 해봤는데 문제가 사라진 것 같다.\
이게 더 좋은 방법인지는 확실하지 않다.\
리눅스 데스크탑을 10년만에 세팅하고 있는 중이다.

## uim-xim 테스트

일단 여기까지 하고 재부팅했다.\
uim-xim을 서비스로 등록하기 전에 작동하는지 확인하고 싶었다.

재부팅하면 터미널을 열어서 uim-xim 을 실행시킨다.

    $ uim-xim

다른 환경, 다른 IM에서 보통 이정도 하면 화면에 언어 선택기가 추가된다.\
uim은 그렇지 않다. 화면에 아무 변화가 없다.

uim-xim은 gnome의 기본 틀을 무시하고 영문 자판에 붙어서 조용히 동작한다.\
약간 당황 스럽지만 그렇다.

## uim 설정

uim 설정을 해야 한글 입력을 테스트할 수 있다.

    $ uim-pref-gtk3

Global settings, Input method deployment 박스,\
Specify default IM 에 체크.

Default input method 에서 Byeoru를 선택.

uim 에 여러 자판을 등록해 두고 바꿔가며 사용하려면\
Input method switching 박스를 설정.\
나는 한글만 사용할 것이므로 필요 없었다.

당황스럽게도 전체 자판들이 모두 enable된 상태에서 시작했다.\
불필요한 자판들은, 제거할 수가 없었다;\
전체 삭제 기능이 없었다.\
하나 삭제하는데 무슨 이유에서인지 엄청 느렸다.\
해서 모두 그대로 두었다.\
그래도 문제는 없었다.

왼쪽 설정창 Global settings 쭉 아래로 가서 Byeoru를 찾는다.\
원하는 대로 한글 입력 방식을 세팅한다.

바로 그 아래 Byeoru Keybinding 1에 가서\
한글입력 모드를 켜고 끌 키를 지정한다.\
on/off 두 필드에 같은 키를 입력하면 된다.

OK를 눌러서 uim-pref를 종료한다.

여기까지 해도 화면에는 아무 변화가 없다.\
하지만 놀랍게도 지금부터 한글을 사용할 수 있다.\
한영변환으로 지정한 키를 한번 누르고 자판을 치니 한글이 입력됐다.

## libgtk-x11-2.0.so.0 에러

gnome 프로그램 몇 개에서는 괜찮았는데\
IntelliJ에서 한글을 입력할 때마다 uim-xim 터미널에 에러가 올라왔다.\
한글 입력은 되는데 로그엔 에러가 뿜는 상태.

라이브러리가 없는 것 같아 gtk2를 설치했다.\
오류는 사라졌지만 이게 맞는 방법인지는 확실하지 않다.

    $ sudo pacman -S gtk2

## uim-xim 서비스 등록

로그인 할 때 uim-xim이 자동 실행되도록 설정을 해야한다.

    ~/.config/systemd/user/uim.service

    [Unit]
    Description=uim daemon
    After=graphical-session.target

    [Service]
    ExecStart=/usr/bin/uim-xim
    Restart=on-abort

    [Install]
    WantedBy=graphical-session.target

Arch 위키에 있는 내용을 Wayland에 맞게 조금 수정했다.

위 설정을 enable 한다.

    $ systemctl enable --user uim.service

재부팅한다.

한영 변환이 잘 되고 한글 입력에도 문제가 없어야 한다.

## 한영상태 표시기

uim-toolbar-* 프로그램들이 있긴 하다.\
하지만 표시하는 것이 더 이상하기도 하고\
그냥 막 실행시키면 크래쉬가 나기도 한다.\
해서 한영상태 툴바는 안 쓰기로 했다.

Arch 문서에 크래쉬를 피하는 방법이 나와있긴 하다.

## IntelliJ 에서 한글 입력이 안 될 경우

gnome 프로그램들에서는 한글입력이 잘 되는데\
IntelliJ에서는 안 되는 현상이 발견되었다.

검색을 해보니 Custom VM 옵션에 아래 내용을 넣어주라는 글이 있었다.

    -Dauto.disable.input.methods=false

하지만 내 경우 이 문제는 아니었다.

uim.service 상태를 확인해 보니 죽어있었다.

    $ systemctl status --user uim

테스트 설정을 잘못해서 uim-xim 이 뜨지도 못한 것 같았다.\
신기한 것은 일단 gnome 프로그램들에서는 한글입력이 된다는 것이었다.\
무슨 조화인지는 모르겠지만 그랬다.

터미널에서 uim-xim을 띄워주니 IntelliJ에서도 한글 입력이 되었다.

## 결론

IBus 보다 좀더 그럴듯한 한글 입력 환경을 만들 수 있었다.

Arch라서 Ubuntu보다 수작업이 많았다.

Xorg가 아니라 Wayland라 옛날 문서들을 거의 활용할 수가 없었다.
