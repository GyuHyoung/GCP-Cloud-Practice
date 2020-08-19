# 클라우드 실습

# GCP VM 인스턴스 만들기

리전 - 서울

머신 - 기본

부팅디스크 - CentOS7

액세스 범위 - 기본 액세스 허용(Cloud API 사용할거면 다른거 클릭)

방화벽 - http, https 둘다 허용

→ 만들기

외부 IP : 이 ip를 통해 원격 접속. 서버를 중지시켰다 켜면 ip 주소 바뀐다 (동적 ip)

SSH → 브라우저 창에서 열기 하면 실행 됨!

but, 세션 타임아웃 자주 걸릴 수 있음

# 간단한 웹서버 만들기

쉘에 `python -m SimpleHTTPServer 8000` 입력

새 브라우저 창에 http://(외부ip):8000 입력 (안됨)

SSH 세션 하나 더 열어서

`curl http://(외부ip):8000` (안됨)

`curl [localhost:8000](http://localhost:8000)` (됨)

*안되는 이유*

방화벽에 의해 막힘. 방화벽은 80, 443 포트만 허용하니까 (http와 https)

[메뉴] → [VPC네트워크] → [방화벽] → Create Firewall Rule

화이트 리스트 만드는 작업!

대상 : 네트워크의 모든 인스턴스

소스필터 : ip 범위 (나머지 옵션은 gcp 내부임)

소스 ip 범위 : [ip4.me](http://ip4.me) 에서 내 ip 복사 or 0.0.0.0/0 (모든 ip?)

지정된 프로토콜 및 포트

tcp : 8000

→ 만들기

이제 웹서버 접속 가능

# GCP에서 나만의 웹서버 생성하기(Marketplace 활용)

marketplace 접속

nginx검색, "NGINX Open Source Certified bu Bitnami" 실행, 배포

visit site → 리다이렉트, 웹서버 보임

# 웹페이지 변경해보기

NGINX VM 인스턴스 찾아서 SSH 브라우저로 열기

`ls /opt/bitnami/nginx/html`

`vi /opt/bitnami/nginx/html/index.html`

원하는거 바꾸기

# 생성된 웹서버를 자동으로 확장하기 Auto Scaling

부하를 분산. Load Scaling

[https://medium.com/google-cloud/scaling-web-app-on-google-compute-engine-d21d6ce3e837](https://medium.com/google-cloud/scaling-web-app-on-google-compute-engine-d21d6ce3e837)

## 우분투 웹서버 생성

[VM 인스턴스]에서 부팅 디스크를 우분투 16.04로 선택, 나머지는 다 기본설정

이름 : web-server

http, https 체크

 [자동화]에 다음 스크립트 복붙

```html
#! /bin/bash
apt update
apt -y install apache2
cat <<EOF > /var/www/html/index.html
<html><body><h1>Hello World</h1>
<p>This page was created from a startup script.</p>
</body></html>
EOF
```

아파치 서버를 설치하고 시작하는 시작스크립트임([https://cloud.google.com/compute/docs/startupscript](https://cloud.google.com/compute/docs/startupscript))

만들기 클릭, 외부 ip로 접속

## 웹서버 스냅샷 만들기

스냅샷 : 서버의 상태를 그대로 저장한다

[메뉴] → [스냅샷] → [스냅샷 만들기]

스냅샷 이름 : snapshot-web-server

멀티리전 체크, 만들기

## VM 이미지 생성하기

스냅샷을 띄우려면 이미지가 필요하다

[메뉴] → [이미지] → [이미지 만들기]

이미지 이름 : image-web-server

소스 : 스냅샷

소스 스냅샷 : snapshot-web-server

다중지역 선택

## 인스턴스 템플릿 생성하기

[메뉴] → [인스턴스 템플릿] → [인스턴스 템플릿 만들기]

부팅 디스크 → 맞춤 이미지 → image-web-server

http, https 선택

## 인스턴스 그룹 만들기

auto scaling 을 정책적으로 관리한다.

리전 : 서울이 아닌 다른 곳 (아시아권에서 선택), ex : 오사카

인스턴스 템플릿 : instance -template-web-server

cpu 사용량 60% 넘으면 자동 확장

축소제어 사용설정 선택 → 인스턴스 수 : 1VMs, 기준시간 : 10분

## Cloud Load Balancer 생성하기