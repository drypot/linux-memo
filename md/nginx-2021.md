# Nginx 2021

간단한 Nginx HTTPS 사이트를 운영하면서 얻은 소소한 경험들에 대한 메모입니다.
Nginx와 Certbot은 도커로 돌렸습니다.

이 글에는 Nginx에 관한 내용만 정리했습니다.
HTTPS, 인증서, Certbot에 관한 내용이 궁금하시면 이 글을 다 보시고 아래 링크를 이어서 보시면 되겠습니다.

[[certbot-2021.md]]

디렉토리 정책등은 취향의 문제이니 참고적으로만 보시면 될 것 같습니다.

## 리포지터리

저는 개발 머신과 서비스 머신에 사용되는 모든 Nginx 설정을 한 리포지터리로 관리하고 있습니다.

<https://github.com/drypot/nginx-conf>

## /data/nginx

Nginx가 동작하는 머신들에 위 리포지터리를 클론해 사용하고 있습니다.

개발 머신에서는 아래 디렉토리에 클론해 두고 씁니다.

    /Users/drypot/projects/nginx-conf    # Mac
    /home/drypot/projects/nginx-conf     # Linux

서비스 머신에서는 아래 디렉토리에 클론해 두고 씁니다.

    /data/nginx/nginx-conf

`/data/nginx` 디렉토리에는 `/data/nginx/nginx-conf` 외에 두 디렉토리를 더 쓰고 있습니다.

    /data/nginx/tmp
    /data/nginx/letsencrypt

`/data/nginx/tmp`에는 업로드 파일이 크게 들어올 경우 Request 전체 덤프가 임시로 저장됩니다.
이 디렉토리를 왜 명시적으로 적었는지는 오래되서 잊었습니다.
추측컨데 스웝 파티션이 있는 디스크하고 분리하려고 그랬던 것 같습니다.

`/data/nginx/letsencrypt`에는 Certbot이 생성한 인증서들이 들어갑니다.

## /data/nginx/nginx-conf/

`nginx-conf` 디렉토리 아래에는 서버별 디렉토리들이 나열되어 있습니다.

    a2/
    a3/
    aws1/
    dev-linux/
    dev-mac/
    common/

`common` 디렉토리에 있는 파일들은 Certbot 메모에서 다루겠습니다.

`aws1/`에 있는 스크립트를 가지고 아래 내용을 이어가겠습니다. 

## aws1/

`aws1/` 디렉토리에는 아마존 라이트세일 인스턴스에서 돌리고 있는 라이브 서버 설정들이 들어있습니다.

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
    
    nginx.conf
    
    sites/

`d-certbot-*` 스트립트들은 Certbot 메모에서 다루겠습니다.

`d-nginx-*.sh`은 Nginx 도커를 운영하는 스크립트들입니다.

`nginx.conf`는 Nginx 도커에 연결할 메일 설정 파일입니다.

`sites/` 아래에 서비스별 Nginx 설정들이 들어갑니다.

## d-nginx-run.sh

Nginx 도커를 실행하는 스크립트입니다.

    #!/bin/bash
    args=(
      --name nginx
      --network=host
      #-p 80:80
      #-p 443:443
      --mount type=bind,source=/data/nginx/nginx-conf/aws1/nginx.conf,target=/etc/nginx/nginx.conf,readonly
      #--mount type=bind,source=/data/nginx/nginx-conf/aws1/enabled,target=/etc/nginx/enabled,readonly
      --mount type=bind,source=/data/nginx/nginx-conf/aws1/sites,target=/etc/nginx/sites,readonly
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

인자 변경을 편하게 하려고 스트립트를 쓰고 있습니다.

위 스크립트를 돌리면 아래와 같은 코맨드가 만들어집니다.

    docker run --name nginx 
    --network=host 
    --mount type=bind,source=/data/nginx/nginx-conf/aws1/nginx.conf,target=/etc/nginx/nginx.conf,readonly 
    --mount type=bind,source=/data/nginx/nginx-conf/aws1/sites,target=/etc/nginx/sites,readonly
    --mount type=bind,source=/data/nginx/tmp,target=/data/nginx/tmp
    --mount type=bind,source=/var/lib/letsencrypt,target=/var/lib/letsencrypt,readonly
    --mount type=bind,source=/data/nginx/letsencrypt,target=/etc/letsencrypt,readonly
    --mount type=bind,source=/data/service,target=/data/service,readonly
    --mount type=bind,source=/data/upload,target=/data/upload,readonly
    -d
    nginx:1.19.7

좀 길고 도커 관련 내용도 있지만 간단하게 언급하면서 지나가겠습니다.

    nginx:1.19.7

끌어다 실행할 도커 이미지 버전입니다.
버전 태깅은 꼭 하시는 것이 좋습니다.
공란으로 두거나 lastest로 하면 추후 이런저런 문제가 발생할 것입니다.

    --name nginx

도커 컨테이너 이름입니다.
이 이름은 다른 스크립트에서 컨테이너를 지정하기 위해 사용합니다.

    --network=host

도커안의 Nginx는 외부의 여러 어플리케이션 서버에 접근할 수 있어야 합니다.
정석적이라면 도커 네트웍을 만들고 모두 거기 접속하는 것이 맞겠지만,
편의상 Nginx를 호스트 네트웍에 바로 물려쓰고 있습니다.

    -d

도커를 데몬으로 돌린다는 말입니다.
테스트할 때는 -d 대신 -it --rm 을 쓰는 것이 편합니다.

    --mount

도커 내부와 호스트 디렉토리를 매핑합니다.
과거엔 -v 인자를 사용했는데 deprecated 되었습니다.
-v 는 디렉토리가 없을 경우 생성을 해버린다고 합니다.
--mount 는 에러를 냅니다. 디버깅에 유리합니다.

호스트 디렉토리를 도커에 연결하는 mount 인자들이 쭉 나옵니다.

    /etc/nginx/nginx.conf
    /etc/nginx/sites

Nginx 메인 설정 파일과 서버별 설정 파일들을 연결합니다.

    /data/nginx/tmp

리퀘스트 덤프가 임시 저장될 곳입니다.

    /var/lib/letsencrypt
    /etc/letsencrypt

Certbot 메모에서 언급하겠습니다.

    /data/service

어플리케이션 서버들이 있는 디렉토리입니다.
어플리케이션의 static 파일들을 서비스하기위해 필요합니다.

    /data/upload

사용자가 업로드한 파일들이 저장된 곳입니다.
저당된 파일은 다시 웹으로 내보낼 수 있어야 합니다.

위 디렉토리들 중 Nginx 설정 관련 디렉토리만 추가 설명하면 될 것 같습니다.
Nginx 설정 설명을 하기 전에 나머지 도커 스크립트 몇 개 더 보고 가겠습니다.

## d-nginx-*

도커 컨테이너를 삭제할 수 있습니다.

    d-nginx-rm.sh

도커 컨테이너 안의 Nginx 설정을 리로드합니다.

    d-nginx-reload.sh

Nginx가 돌고 있는 컨테이너 안에 쉘을 띄웁니다.

    d-nginx-shell.sh

컨테이너가 재부팅 이후 자동 실행되도록 합니다.

    d-nginx-service.sh

도커 컨테이너 재실행은 도커가 모두 관리합니다.
systemd 로 관리하지 않아도 됩니다.

## nginx.conf

`nginx.conf`는 Nginx 메인 설정파일입니다.
 
    http {
        include /etc/nginx/sites/enabled/*.conf;
    }

이 설정 파일에서 제가 언급할 만한 것은 `include` 딱 한 줄입니다.
다른 설정들은 Nginx 레퍼런스 문서를 보시면 될 것 같습니다.

Nginx는 http 블럭 아래 여러 server 블럭을 갖습니다.
메인 설정파일에는 server 블럭을 두지 않는 것이 깔끔합니다.

## sites/

sites 디렉토리에는 서비스별 설정파일들이 개별적으로 존재합니다.

    default.conf
    
    raysoda-m.conf
    raysoda.conf
    ...

    enabled/

sites 디렉토리에 있는 모든 사이트들이 활성화되는 것은 아닙니다.
사이트는 필요에 따라 올리고 내려야할 필요가 있습니다.
예로, 사이트를 정비하는 동안에는 정규 설정을 빼고 점검 공지 사이트를 올려야합니다.

사이트를 활성화 하는 방법은 enabled 디렉토리를 만들고 여기 상대경로 심볼릭 링크를 만드는 것입니다.

예로 raysoda.conf 를 활성화하고 싶다면 아래처럼 합니다.

    $ ln -sr raysoda.conf enabled

`default.conf`는 기본적으로 항상 활성화 해둬야합니다.
Certbot 메모에서 언급하겠습니다.

## 마무리

이정도면 일단 기본적인 Nginx 운영 틀은 설명된 것 같습니다.
HTTPS 관련 설정이 필요하면 아래 글을 이어서 보시면 됩니다.

[[certbot-2021.md]]
