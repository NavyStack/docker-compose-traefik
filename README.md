# docker-compose-traefik

## 간단 정리

1. 도커를 설치합니다.
2. docker-compose.yml 파일을 수정합니다.
3. (VScode 기준) 설명 우선 Ctrl + H로 `example.com` 을 본인의 도메인으로 수정합니다.
4. Ctrl + H로 `example-`을 본인의 별칭으로 수정합니다. 저는 도메인 기준으로 알기 편하게 tld를 제외하는 식으로 마킹했습니다. 또한 컨테이너 이름을 알기 쉽게 바꾸기 위함입니다.
5. 현재 디렉토리에 `docker - nginx`로 이동합니다. 폴더 명도 도메인 기준으로 수정해 줍니다.
6. example.com.conf 파일을 열고 upsteam php를 `"4번에서 변경한 별칭 + wordpress"`로 수정합니다 (기존은 example-wordpress 이었음)

7. docker-compose-traefik의 디렉토리로 이동해서 `docker compose up -d` 하면 Traefik, wordpress php-fpm, nignx, mysql, redis 그리고 DB 백업용 사이드카 컨테이너가 실행됩니다.

## (기타) CentOS, Oracel linux 등의 Docker 설치

`yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

`yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

`systemctl enable docker.service`

`systemctl enable containerd.service`

`systemctl start docker`

`chown root:docker /var/run/docker.sock`

(`usermod -aG docker $USER`가 기본 구조임 $USER를 사용하면 현재 계정이 추가됩니다. 따라서 아래와 같이 적절하게 유저 이름을 바꿔야합니다.)

유저 이름이 opc라면 아래와 같이 입력

`usermod -aG docker opc`

`reboot`

## 도커 네트워크 및 바인드할 경로 미리 만들기

`docker network create traefik-network`

`docker network create examlple-network`

`docker network create backup-network`

docker compose의 network를 미리 만들어줍니다.
그리고 volume을 생성합니다. (device: 다음에 적혀있는 경로 채로 생성하면 됩니다.)

/home/opc/docker-compose-traefik/DATA/traefik-certificates 등등등 있으면,

`mkdir -p /home/opc/docker-compose-traefik/DATA/traefik-certificates`

`mkdir -p /home/opc/docker-compose-traefik/BACKUPS/data-backups`  
이런식으로 전부 생성하면 됩니다.

이는 rclone 등의 방법으로 백업을 용이하게 하기 위해서 폴더를 분리했습니다.

### SELinux 관련 설정하기 (유저 이름 opc의 홈 폴더에 git을 했다고 가정)

`semanage fcontext -a -t svirt_sandbox_file_t '/home/opc/docker-compose(/.*)?'`
`semanage fcontext -a -t svirt_sandbox_file_t '/home/opc/docker-compose_Databases(/.*)?'`

`restorecon -Rv /home/opc/docker-compose-traefik`
`restorecon -Rv /home/opc/docker-compose-traefik_Databases`

## 방화벽 열기

`firewall-cmd --add-service=http --add-service=https --permanent`

`firewall-cmd --add-port=443/udp --permanent`

`firewall-cmd --reload`

`firewall-cmd --list-all`

전부 진행 했으면 `docker-compose-traefik`의 경로로 이동해서 `docker compose up -d`
