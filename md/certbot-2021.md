# Certbot 2021

도커로 Nginx와 Certbot을 돌리고 있습니다. 이 문서는 이에 관한 설명입니다.

두 편의 글로 구성되어 있습니다.
이 글에서는 HTTPS 인증서, Certbot에 대해서 설명합니다.
Nginx 기본 내용은 아래 링크에 있습니다.

[nginx-2021.md](nginx-2021.md)

아래 내용을 보기 전에 위 Nginx 글을 보고 오시면 좋습니다.

## 수정 기록

초판: 2021-03-04\
수정: 2021-06-18, 개발용 설정과 라이브용 설정을 분리

## Nginx Conf 리포지터리

<https://github.com/drypot/nginx-conf-aws1> \
<https://github.com/drypot/nginx-conf-mac>

라이브 리눅스 머신과 맥 개발 머신에 사용하고 있는 Nginx 설정 리포지터리입니다. 아래 내용은 이 리포지터리에 대한 설명입니다.

## Let's Encrypt

<https://letsencrypt.org/>

HTTPS 보안 통신을 하려면 서버쪽에 퍼블릭키, 프라이빗키 한 세트가 필요합니다.
예전엔 이 키를 발급받기가 꽤 불편했습니다.
그러다 Let's Encrypt 가 생기면서 무료로 키를 받을 수 있게 됐습니다.

## Certbot

<https://certbot.eff.org/docs/>

Certbot은 Let's Encrypt의 키를 발급 받는데 사용하는 코멘드라인 유틸리티입니다.
코맨드 하나로 구성되어 있지만 디펜던시가 커서 도커로 땡겨 쓰는 것이 깔끔합니다.

Certbot은 두 가지 기능을 합니다. 하나는 키를 받아서 로컬에 저장하는 것입니다.
또 하나는 Nginx 설정을 수정해서 키를 Nginx에 연결하는 것입니다.

Nginx 자동 설정 기능이 잘 작동하면 좋겠지만 실제 사용해 보니 문제가 있었습니다.
첫 인증서 발급은 성공했지만 Certbot이 만들어준 설정으로는 인증서 갱신이 되지 않았습니다.
언젠간 고쳐지겠지만 Nginx 설정을 심하게 바꿔서 좀 꺼려지는 부분도 있습니다.

해서 Certbot으로는 인증서만 받고 수작업으로 Nginx에 심는 방식을 사용했습니다.

## 도메인 소유 증명

키를 받으려면 도메인 소유를 증명해야 합니다.
몇 가지 방법이 있는데 내 웹 서버 디렉토리에 받아쓰는 방식을 사용했습니다.

Certbot이 Let's Encrypt(이하 LE)에 인증 요청을 하면 LE가 내 웹 사이트 아래 적을 문구를 줍니다.
Certbot은 이 문구를 내 웹사이트 아래에 적습니다. LE는 80포트로 내 사이트에 접속해서 문구를 확인하고 인증서를 발급합니다.

여기서 내 사이트의 80 포트는 Nginx의 관할 영역입니다.
해서 Certbot과 Nginx가 이 부분에서 같이 움직여야 합니다.
같은 디렉토리가 Certbot 설정에도 나오고 Nginx 설정에도 나옵니다.

    /var/lib/letsencrypt

Certbot 문서에는 위 디렉토리를 이 용도로 사용하라고 나옵니다. 해서 호스트, Nginx 도커, Certbot 도커, 모두 같은 패스를 매핑해서 사용했습니다.
 
    --mount type=bind,source=/var/lib/letsencrypt,target=/var/lib/letsencrypt,readonly

Nginx 도커 설정중.

    --mount type=bind,source=/var/lib/letsencrypt,target=/var/lib/letsencrypt

Certbot 도커 설정중.

우리는 이 디렉토리 내용을 볼 필요가 없습니다.
Certbot이 임시 데이터를 잠깐 썼다 지웁니다.

## 인증서 디렉토리

    /etc/letsencrypt

발급받은 인증서는 위 디렉토리에 쌓입니다.
이 디렉토리는 Certbot 이 생성하고 관리합니다.

인증서들은 재발급 받을 수 있습니다.
안전하게 보관할 필요는 없습니다.

이 디렉토리도 Nginx와 공유해야 하고 아래 나올 몇 가지 수작업 설정을 해야 해서
호스트에 매핑해야 합니다.

    /data/nginx/letsencrypt

저는 이 디렉토리에 매핑했습니다.
    
    --mount type=bind,source=/data/nginx/letsencrypt,target=/etc/letsencrypt,readonly


Nginx 도커 설정중.

    --mount type=bind,source=/data/nginx/letsencrypt,target=/etc/letsencrypt

Certbot 도커 설정중.

## 인증서 발급 받기 / Nginx

    server {
      listen 80 default_server;
      server_name _;
    
      location ^~ /.well-known/ {
        allow all;
        alias /var/lib/letsencrypt/.well-known/;
        default_type "text/plain";
        try_files $uri =404;
      }
    
      location / {
        return 301 https://$host$request_uri;
      }
    }

`sites/default.conf`입니다.
이 파일은 Certbot 서비스용 서버 블럭을 정의하고 있습니다.

인증서는 Certbot으로 받지만
받아쓰기 디렉토리를 외부로 노출할 Nginx 설정이 필요합니다.
해서 `default.conf`는 항상 활성화되어 있어야 합니다.

받아쓰기를 확인하기 위해 80 포트로 리퀘스트가 들어오면
`/var/lib/letsencrypt/.well-known/` 내용을 보여줍니다.
그 외 리퀘스트는 모두 `https://`로 리다이렉트합니다.

80 포트 정의는 여기 한 곳이어야 합니다.
다른 서버들은 모두 HTTPS 443 포트를 사용하고 있는 것이 합당합니다.

인증을 해야하는 사이트가 추가되더라고 `default.conf`를 수정할 필요는 없습니다.

    $ cd sites
    $ ln -s ../default.conf enabled

`default.conf`를 활성화하는 방법은 위와 같습니다. 링크 생성후 설정을 리로드해야 합니다.


## 인증서 발급 받기 / Certbot

    #!/bin/bash
    args=(
      --name certbot-new
      -it --rm 
      --mount type=bind,source=/data/nginx/letsencrypt,target=/etc/letsencrypt
      --mount type=bind,source=/var/lib/letsencrypt,target=/var/lib/letsencrypt
      certbot/certbot:v1.12.0
      certonly
      --webroot
      -w /var/lib/letsencrypt/
    )
    docker run "${args[@]}" "$@"

도커 인자가 복잡하게 들어가서 스크립트를 만들었습니다.
인증서는 `d-certbot-new.sh` 스크립트로 발급받습니다.

    $ bin/d-certbot-new.sh -d raysoda.com -d www.raysoda.com -d file.raysoda.com

스크립트는 위와 같은 형식으로 사용합니다. 실행시키면 몇 가지 입력 사항을 접수한 후 인증서를 만들어 로컬에 저장합니다.

`-d` 로 인증서에 포함할 모든 도메인 이름을 나열합니다.
당연히 DNS에 도메인을 현재 호스트 IP로 매핑해 두셨어야 합니다.


    docker run
    --name certbot-new
    -it --rm
    --mount type=bind,source=/data/nginx/letsencrypt,target=/etc/letsencrypt
    --mount type=bind,source=/var/lib/letsencrypt,target=/var/lib/letsencrypt
    certbot/certbot:v1.12.0

    certonly
    --webroot -w /var/lib/letsencrypt/
    -d raysoda.com
    -d www.raysoda.com
    -d file.raysoda.com

스크립트를 돌렸을 때 생성되는 실행 코맨드는 위와 같습니다.

각 인자들을 간단히 보겠습니다.

    certbot/certbot:v1.12.0

끌어다 실행할 Certbot 이미지 버전입니다.
버전 태깅은 꼭 하는 것이 좋습니다.
공란으로 두거나 lastest로 하면 추후 이런저런 문제가 발생합니다.

    --name certbot-new

도커 컨테이너 이름입니다. 잠깐 실행되었다 살아질 것이니 큰 의미는 없습니다.

    -it --rm

인터렉티브하게 실행했다가 실행이 끝나면 컨테이너를 삭제하란 뜻입니다.

    --mount ...
    --mount ...

Nginx와 두 디렉토리를 공유해야 합니다.
자세한 설명은 위에 했습니다.

여기까지가 도커 인자들입니다.

    certonly
    --webroot
    -w /var/lib/letsencrypt/
    -d raysoda.com
    -d www.raysoda.com
    -d file.raysoda.com

certonly 부터는 Certbot 인자입니다.

    certonly

인증서를 새로 발급받겠다는 뜻입니다.
이 자리에 올 수 있는 명령들 몇 가지가 있습니다.
`renew`, 인증서를 재발급 받겠다.
`install`, 웹 서버에 인증서를 심고 싶으니 웹 서버 설정을 수정하라.
`run`, 인증서를 발급받고 웹 서버에 심는 두 작업을 동시에 하라.

위에서 말했다시피 저는 `install`과 `run`을 사용하지 않고 있습니다.
자동생성된 Nginx 설정에 문제가 좀 있습니다.
해서 `certonly`와 `renew`만 씁니다.

    --webroot

인증서 발급 받기 위해 도메인 소유증명을 하는 방법이 몇 가지 있습니다.
`--webroot`는 웹 사이트에 받아쓰는 방법을 사용하겠다는 뜻입니다.

    -w /var/lib/letsencrypt/

웹 사이트 받아쓰기할 디렉토리를 지정합니다.
공식 문서에 여기를 쓰라고 권장하고 있으니 그냥 그대로 씁니다.

    -d xxx.com

인증서에 포함할 도메인들을 쭉 나열합니다.


## 인증서 사용하기

    server {
      server_name raysoda.com;
      root /data/service/raysoda/raysoda/public;
      client_max_body_size 48m;
      client_body_temp_path /data/nginx/tmp;
    
      location / {
        try_files $uri @app;
      }
    
      location @app {
        proxy_pass http://localhost:8050;
        proxy_set_header Host $http_host;
      }
    
      listen 443 ssl;
      ssl_certificate /etc/letsencrypt/live/raysoda.com/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/raysoda.com/privkey.pem;
      include /etc/letsencrypt/options-ssl-nginx.conf;
      ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    }

인증서를 받았으면 이를 사용하는 Nginx 서버 블럭을 만들어야 합니다.
`certbot install`로 자동으로 되어야 하는데 아직 문제가 좀 있습니다.
수작업을 좀 합니다. 위는 수작업 결과의 예입니다.

HTTPS 관련 설정은 아래 5줄입니다.

      ssl_certificate /etc/letsencrypt/live/raysoda.com/fullchain.pem;

Certbot이 받아 놓은 퍼블릭키를 지정합니다.
저 위치 가보면 비슷한 것이 있을 겁니다.

      ssl_certificate_key /etc/letsencrypt/live/raysoda.com/privkey.pem;

Certbot이 받아 놓은 프라이빗키를 지정합니다.

      include /etc/letsencrypt/options-ssl-nginx.conf;
      ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

이 두 줄은 추가 설명이 좀 필요합니다.

## options-ssl-nginx.conf

HTTPS 설정을 마무리하려면 좀 복잡한 Nginx 설정을 우겨 넣어야 합니다.
이 내용은 일반 웹 서버 관리자가 이해하기 어려운 것들입니다.

이 내용을 Certbot이나 Nginx에서 제공해줬으면 좋겠는데 항상 그럴 수 있는 것이 아닙니다.
Nginx 버전에 따라, 서버에 설치된 SSL 버전에 따라,
대상으로 할 사용자 브라우저 범위에 따라 설정이 달라집니다.
다행이 권장 옵션을 만들어주는 곳이 있습니다.

<https://ssl-config.mozilla.org/>

여기서 인자를 넣으면 옵션들을 생성해 줍니다. 많은 인자들이 나열되는데 Certbot 리포를 보면 이걸 대략 결정해 둔 것이 없진 않습니다.

<https://github.com/certbot/certbot/tree/master/certbot-nginx>

여기를 타고 들어가 좀 헤메다 보면 `options-ssl-nginx.conf` 파일이 보입니다.
몇 년에 한번씩 업데이트하는 것 같습니다.
이 파일을 로컬에 저장해서 인증서들이 모여있는 `letsenrypt` 디렉토리 상단에 넣습니다.
이 파일은 한번 저장해두고 모든 https 서버 블럭에서 인클루드하면 됩니다.

    /data/nginx/letsencrypt/options-ssl-nginx.conf

제 설정에서는 위 위치에 들어갑니다.

    $ sudo chown root:root /data/nginx/letsencrypt/options-ssl-nginx.conf
    $ sudo chmod 644 /data/nginx/letsencrypt/options-ssl-nginx.conf

파일 소유권, 퍼미션 수정이 필요하면 적당히 해줍니다.

이 파일을 Certbot이 미리 좀 만들어주면 좋겠는데 `install` 명령을 실행했을 때만 만듭니다.
포럼에 보니 `certonly` 사용자들을 위해 이것만 자동으로 만들어 달라는 요청이 있긴 합니다. 구현될지는 미지수.

<https://community.letsencrypt.org/t/generating-options-ssl-nginx-conf-and-ssl-dhparams-in-certonly-mode/136272>

## ssl_dhparam

HTTPS 설정중 `ssl_dhparam`을 적는 부분이 있습니다.
dhparam을 로컬에 생성하고 파일 패스를 걸어줘야 합니다.

    curl https://ssl-config.mozilla.org/ffdhe2048.txt > /path/to/dhparam

dhparam을 만드는 방법은 위 모잘라 사이트 인자들 사이에 코멘트로 적혀있습니다.
오늘 기준은 위에처럼 하라고 되어 있습니다.
이것도 한번 만들면 모든 서버 블럭에서 공용으로 쓰면 됩니다.

    /data/nginx/letsencrypt/ssl-dhparams.pem

제 경우 위 위치에 만들었습니다.

    $ sudo chown root:root ssl-dhparams.pem
    $ sudo chmod 644 ssl-dhparams.pem

파일 소유권, 퍼미션 수정이 필요하면 적당히 해줍니다.

## 서버 활성화

    $ ln -sr raysoda.conf enabled

Nginx 서버 블럭 설정을 완성했으면 `sites` 디렉토리에 들어가 활성화합니다.

    $ sudo docker exec -it nginx bash
    # nginx -t

Nginx 도커로 들어가서 설정을 테스트합니다.

    # nginx -s reload

이상이 없으면 설정을 리로드합니다.

브라우저로 접속해 확인합니다.

## 인증서 재발급

    $ bin/d-certbot-renew-cron.sh

인증서 재발급 스크립트입니다.

    #!/bin/bash
    args=(
      --name certbot-renew
    # -it 
      --rm 
      --mount type=bind,source=/data/nginx/letsencrypt,target=/etc/letsencrypt
      --mount type=bind,source=/var/lib/letsencrypt,target=/var/lib/letsencrypt
      certbot/certbot:v1.12.0
      renew
    )
    docker run "${args[@]}" "$@"

내용은 위와 같습니다.

인터렉티브하게 돌릴 것이 아니므로 `-it` 인자는 넣지 않습니다.

## 인증서 재발급 자동화

    $ sudo crontab -e  # root 용을 수정
    $ crontab -e       # 일반 사용자용을 수정

인증서 재발급 자동화는 cron으로 하면 무난합니다.

스크립트 실행에 필요한 퍼미션에 따라 root/일반 cron을 구분해서 사용합니다.
제 경우 모든 서버 스크립트를 일반 계정으로 돌리고 있어서 `crontab -e`를 썼습니다.

    0 5 * * 2 /data/nginx/nginx-conf-aws1/bin/d-certbot-renew-cron.sh > /data/cron/certbot.log 2>&1
    30 5 * * 2 /data/nginx/nginx-conf-aws1/bin/d-nginx-reload-cron.sh > /data/cron/nginx.log 2>&1

crontab 내용입니다. 일주일에 한번 화요일 새벽에 스크립트를 돌리도록 설정했습니다.

화요일 새벽 5시에 인증서를 업데이트합니다.
화요일 새벽 5시 30분에 nginx에서 갱신된 인증서를 리로드합니다.

`> /data/cron/certbot.log` 부분은 스크립트 실행시 적당한 곳에 로그를 남기는 것입니다.

`2>&1`는 2번 stderr 파일핸들 출력을 1번으로 함께 보내라는 뜻입니다.
