# Nginx 2021

도커로 Nginx와 Certbot을 돌리고 있습니다. 이 문서는 이에 관한 설명입니다.

두 편의 글로 구성되어 있습니다.
이 글에서는 Nginx 설정에 대해서 설명합니다.
HTTPS 인증서, Certbot에 관한 내용은 아래 링크에 있습니다.

[certbot-2021.md](certbot-2021.md)

## 수정 기록

초판: 2021-03-04\
수정: 2021-06-18, 개발용 설정과 라이브용 설정을 분리

## Nginx Conf 리포지터리

<https://github.com/drypot/nginx-conf-aws1> \
<https://github.com/drypot/nginx-conf-mac>

라이브 리눅스 머신과 맥 개발 머신에 사용하고 있는 Nginx 설정 리포지터리입니다. 아래 내용은 이 리포지터리에 대한 설명입니다.

## 클론 위치

    /data/nginx/nginx-conf-aws1             # aws1 서비스 머신
    /Users/drypot/projects/nginx-conf-mac   # Mac 개발 머신
    /home/drypot/projects/nginx-conf-linux  # Linux 개발 머신

각 머신별 설정 리포지터리를 클론한 위치입니다.

    /data/nginx/tmp
    /data/nginx/letsencrypt

`/data/nginx` 디렉토리에는 위 서브디렉토리가 두 개가 더 들어갑니다.

`/data/nginx/tmp`는 업로드 파일이 크게 들어올 경우 Request 덤프를 임시 저장하는데 사용합니다.

`/data/nginx/letsencrypt`에는 Certbot이 생성한 인증서들이 들어갑니다.

## 루트 디렉토리

    nginx.conf

클론의 루트 디렉토리에는 유의미한 파일이 한 개 있습니다. 
nginx의 메인 설정파일입니다.
추후 도커를 경유하여 nginx 에 연결됩니다.


    bin/

    d-certbot-new-drypot.sh
    d-certbot-new-osoky.sh
    d-certbot-new-rapixel.sh
    d-certbot-new-raysoda.sh
    d-certbot-new-sleek.sh
    d-certbot-new.sh
    d-certbot-renew.sh
    
    d-nginx-reload.sh
    d-nginx-rm.sh
    d-nginx-run.sh
    d-nginx-service.sh
    d-nginx-shell.sh

`bin` 디렉토리에는 도커용 쉘 스크립트들이 들어있습니다.

    sites/
    sites/enabled

`sites` 디렉토리 아래에는 서비스별 설정들이 들어갑니다.

## d-nginx-run.sh 스크립트

    #!/bin/bash
    args=(
      --name nginx
      --network=host
      #-p 80:80
      #-p 443:443
      --mount type=bind,source=/data/nginx/nginx-conf-aws1/nginx.conf,target=/etc/nginx/nginx.conf,readonly
      --mount type=bind,source=/data/nginx/nginx-conf-aws1/sites,target=/etc/nginx/sites,readonly
      --mount type=bind,source=/data/nginx/tmp,target=/data/nginx/tmp
      --mount type=bind,source=/var/lib/letsencrypt,target=/var/lib/letsencrypt,readonly
      --mount type=bind,source=/data/nginx/letsencrypt,target=/etc/letsencrypt,readonly
      --mount type=bind,source=/data/service,target=/data/service,readonly
      --mount type=bind,source=/data/upload,target=/data/upload,readonly
    #  -it --rm
      -d
      nginx:1.19.7
    #  nginx-debug -g 'daemon off;'
    )
    docker run "${args[@]}"

Nginx 도커를 실행하는 스크립트입니다.
인자 변경을 편하게 하려고 bash 스크립트를 쓰고 있습니다.
위 스크립트를 돌리면 아래와 같은 코맨드가 만들어집니다.

    docker run --name nginx 
    --network=host 
    --mount type=bind,source=/data/nginx/nginx-conf-aws1/nginx.conf,target=/etc/nginx/nginx.conf,readonly 
    --mount type=bind,source=/data/nginx/nginx-conf-aws1/sites,target=/etc/nginx/sites,readonly
    --mount type=bind,source=/data/nginx/tmp,target=/data/nginx/tmp
    --mount type=bind,source=/var/lib/letsencrypt,target=/var/lib/letsencrypt,readonly
    --mount type=bind,source=/data/nginx/letsencrypt,target=/etc/letsencrypt,readonly
    --mount type=bind,source=/data/service,target=/data/service,readonly
    --mount type=bind,source=/data/upload,target=/data/upload,readonly
    -d
    nginx:1.19.7

간단하게 언급하면서 지나가겠습니다.

    nginx:1.19.7

끌어다 실행할 도커 이미지 버전입니다.
버전 태깅은 꼭 하는 것이 좋습니다.
공란으로 두거나 lastest로 하면 추후 이런저런 문제가 발생합니다.

    --name nginx

도커 컨테이너 이름입니다.
다른 스크립트에서 컨테이너를 지정하기 위해 사용합니다.

    --network=host

도커안의 Nginx는 외부의 여러 어플리케이션 서버에 접근할 수 있어야 합니다.
정석적이라면 도커 네트웍을 만들고 모두 거기 접속하는 것이 맞겠지만,
편의상 Nginx를 호스트 네트웍에 바로 물려쓰고 있습니다.

    -d

도커를 데몬으로 돌린다는 말입니다.
테스트할 때는 `-d` 대신 `-it --rm` 을 쓰는 것이 편합니다.

    --mount

호스트 디렉토리를 도커에 연결하는 mount 인자들이 쭉 나옵니다.
`--mount` 는 도커 내부 디렉토리와 호스트 디렉토리를 매핑합니다.
과거엔 `-v` 인자를 사용했는데 deprecated 되었습니다.
`-v` 는 디렉토리가 없을 경우 이를 생성해서 디버깅을 어렵게 만든다고 합니다.
`--mount` 는 에러를 내고 실행을 중지합니다.

    --mount ... /etc/nginx/nginx.conf
    --mount ... /etc/nginx/sites

Nginx 메인 설정 파일과 서버별 설정 파일들을 연결합니다.

    --mount ... /data/nginx/tmp

리퀘스트 덤프가 임시 저장될 곳입니다.

    --mount ... /var/lib/letsencrypt
    --mount ... /etc/letsencrypt

Certbot 메모에서 언급하겠습니다.

    --mount ... /data/service

어플리케이션 서버들이 있는 디렉토리입니다.
어플리케이션의 static 파일들을 서비스하기 위해 필요합니다.

    --mount ... /data/upload

사용자가 업로드한 파일들이 저장되어 있습니다.

위 디렉토리들 중 Nginx 설정 디렉토리만 추가 설명하면 될 것 같습니다.

Nginx 설정을 설명하기 전에 나머지 도커 스크립트 몇 개 더 보고 가겠습니다.

## d-nginx-* 스크립트

    d-nginx-rm.sh

도커 컨테이너를 삭제합니다.

    d-nginx-reload.sh

도커 컨테이너 안의 Nginx 설정을 리로드합니다.

    d-nginx-shell.sh

Nginx가 돌고 있는 컨테이너 안에 쉘을 띄웁니다.

    d-nginx-service.sh

재부팅 후 컨테이너가 자동 실행되도록 세팅합니다.

도커 컨테이너 실행은 도커가 모두 관리합니다.
systemd 로 하지 않아도 됩니다.

## nginx.conf 파일

    http {
        ...

        include /etc/nginx/sites/enabled/*.conf;
    }

`nginx.conf`는 Nginx 메인 설정파일입니다.

메인 설정 파일은 `http` 블럭 아래 여러 `server` 블럭을 갖습니다.
메인 설정 파일에는 `server` 블럭을 두지 않는 것이 깔끔합니다.
대신 위에 처럼 `include`로 처리합니다. 다른 설정들은 Nginx 레퍼런스 문서를 보시면 될 것 같습니다.


## sites 디렉토리

    sites/

    default.conf
    
    raysoda-m.conf
    raysoda.conf
    osoky-m.conf
    osoky.conf
    ...

    sites/enabled/
    ...

sites 디렉토리에는 서비스별 설정 파일들이 들어갑니다.
sites 디렉토리에 있는 모든 사이트들이 활성화되는 것은 아닙니다.
사이트는 필요에 따라 올리고 내려야할 필요가 있습니다.
예로, 사이트를 정비하는 동안에는 정규 설정을 빼고 점검 공지 사이트를 올려야합니다.

사이트를 활성화 하는 방법은 enabled 디렉토리를 만들고 상대경로 심볼릭 링크를 만드는 것입니다. Nginx 메인 설정에서는 이 enabled 디렉토리를 인클루드합니다.

`enabled` 디렉토리들은 `.gitignore` 에 정의되어 있어서 리포에 올라가지 않습니다.

    $ ln -s ../default.conf enabled

`default.conf` 를 활성화합니다.

HTTPS를 쓰려면 `default.conf`는 기본적으로 항상 활성화해둬야 합니다.
Certbot 메모에서 언급하겠습니다.

    $ rm enabled/raysoda.conf

링크를 삭제해서 사이트를 비활성화할 수 있습니다.

## Certbot

이정도면 일단 기본적인 Nginx 운영 틀은 설명된 것 같습니다.

HTTPS 관련 설정이 아래 글에 이어집니다.

[certbot-2021.md](certbot-2021.md)
