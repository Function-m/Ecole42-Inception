# Ecole42-Inception

## 목차

1. [프로젝트 개요](#프로젝트-개요)
2. [기술 스택](#기술-스택)
3. [구현 사항](#구현-사항)
   - [Docker 설정](#docker-설정)
   - [NGINX 설정](#nginx-설정)
   - [WordPress + php-fpm 설정](#wordpress--php-fpm-설정)
   - [MariaDB 설정](#mariadb-설정)
   - [Volume 설정](#volume-설정)
   - [Network 설정](#network-설정)
   - [도메인 및 환경 변수 설정](#도메인-및-환경-변수-설정)
4. [실행 방법](#실행-방법)
5. [트러블 슈팅](#트러블-슈팅)
6. [참고 자료](#참고-자료)

## 프로젝트 개요

이 프로젝트는 Docker와 Docker Compose를 사용하여 소규모 인프라를 구축하는 것을 목표로 합니다. NGINX, WordPress + php-fpm, MariaDB를 독립적인 Docker 컨테이너로 구성하고, WordPress 관련 데이터는 Volume을 통해 영속적으로 관리합니다. 모든 컨테이너는 Docker Network를 통해 상호 연결되며, NGINX를 통해 외부에서 접근할 수 있도록 설정합니다.

## 기술 스택

* Docker
* Docker Compose
* NGINX (TLSv1.2 또는 TLSv1.3)
* WordPress + php-fpm
* MariaDB
* Alpine 또는 Debian (Debian 사용됌)
* `.env` 파일을 통한 환경 변수 관리

## 구현 사항

###   Docker 설정

* 모든 컨테이너는 Debian을 기반으로 빌드됩니다.
* `docker-compose.yml` 파일을 통해 각 서비스를 정의하고, Dockerfile을 사용하여 이미지를 빌드합니다.
* `latest` 태그는 사용하지 않습니다.

###   NGINX 설정

* NGINX 컨테이너는 443 포트를 통해 외부 요청을 처리하며, TLSv1.2 또는 TLSv1.3 프로토콜을 사용하여 보안 연결을 제공합니다.
* 적절한 NGINX 설정을 통해 WordPress로의 요청을 프록시합니다.

###   WordPress + php-fpm 설정

* WordPress와 php-fpm은 별도의 컨테이너로 실행되며, php-fpm은 WordPress의 요청을 처리합니다.
* WordPress는 MariaDB 데이터베이스와 연동되도록 설정됩니다.

###   MariaDB 설정

* MariaDB 컨테이너는 WordPress 데이터를 저장하는 데 사용됩니다.
* WordPress에서 사용할 데이터베이스와 사용자를 생성하고, 적절한 권한을 부여합니다.
* 관리자 계정의 사용자 이름에는 "admin", "Admin", "administrator", "Administrator"를 포함할 수 없습니다.

###   Volume 설정

* WordPress 웹사이트 파일과 MariaDB 데이터베이스 파일은 각각 별도의 Docker Volume에 저장하여 데이터의 영속성을 보장합니다.
* 이 Volume들은 호스트 시스템의 `/home/login/data` 디렉토리에서 접근할 수 있습니다. (여기서 `login`은 사용자 로그인 이름입니다.)

###   Network 설정

* Docker Network를 생성하여 모든 컨테이너가 서로 통신할 수 있도록 합니다.
* `network_mode: "host"` 또는 `--link` 또는 `links:`는 사용하지 않습니다.

###   도메인 및 환경 변수 설정

* 도메인 이름은 `login.42.fr`로 설정하여 로컬 IP 주소를 가리키도록 합니다. (여기서 `login`은 사용자 로그인 이름입니다.)
* 비밀번호, 데이터베이스 이름 등 중요한 정보는 `.env` 파일에 저장하고, Dockerfile 내부에 직접 포함하지 않습니다.
* 컨테이너가 비정상 종료되더라도 자동으로 재시작되도록 설정합니다.
* 컨테이너 시작 시 `tail -f`, `bash`, `sleep infinity`, `while true`와 같은 무한 루프를 실행하는 명령어는 사용하지 않습니다.

##   실행 방법

1.  **가상 머신 설정**: VirtualBox 또는 VMware 등의 가상화 소프트웨어를 사용하여 가상 머신을 설정합니다.
2.  **Docker 및 Docker Compose 설치**: 가상 머신에 Docker와 Docker Compose를 설치합니다.
3.  **프로젝트 복제**: Git을 사용하여 프로젝트 레포지토리를 복제합니다.
4.  **.env 파일 설정**: `.env.example` 파일을 복사하여 `.env` 파일을 생성하고, 필요한 환경 변수(예: 데이터베이스 비밀번호, 도메인 이름)를 적절하게 설정합니다.
5.  **Makefile 실행**: 프로젝트 루트 디렉토리에서 `make build` 명령어를 실행하여 Docker 이미지를 빌드하고, `make up` 명령어를 실행하여 컨테이너를 시작합니다.
6.  **웹 브라우저 접근**: 웹 브라우저에서 설정한 도메인 이름(`login.42.fr`)으로 접속하여 WordPress가 정상적으로 실행되는지 확인합니다.

## 트러블 슈팅

* **NGINX 설정 오류**: NGINX 설정 파일의 문법 오류로 인해 NGINX 컨테이너가 정상적으로 시작되지 않는 문제가 발생했습니다. `nginx -t` 명령어를 사용하여 설정 파일의 문법을 확인하고, 오류를 수정하여 해결했습니다.
* **데이터베이스 연결 문제**: WordPress 컨테이너에서 MariaDB 컨테이너로의 연결이 실패하는 문제가 발생했습니다. `.env` 파일에 설정된 데이터베이스 호스트, 사용자 이름, 비밀번호 등이 올바른지 확인하고, `docker network inspect` 명령어를 사용하여 컨테이너 간의 네트워크 연결을 확인하여 해결했습니다.
* **파일 권한 문제**: WordPress 파일 업로드 시 권한 문제로 인해 업로드가 실패하는 문제가 발생했습니다. Docker Volume의 권한을 변경하여 WordPress 컨테이너에서 파일에 접근할 수 있도록 설정하여 해결했습니다.

## 참고 자료

* Docker Documentation: [https://docs.docker.com/](https://docs.docker.com/)
* Docker Compose Documentation: [https://docs.docker.com/compose/](https://docs.docker.com/compose/)
* NGINX Documentation: [https://nginx.org/en/docs/](https://nginx.org/en/docs/)
* WordPress Documentation: [https://wordpress.org/documentation/](https://wordpress.org/documentation/)
* MariaDB Documentation: [https://mariadb.com/kb/en/](https://mariadb.com/kb/en/)
* Alpine Linux: [https://www.alpinelinux.org/](https://www.alpinelinux.org/)
* Debian Linux: [https://www.debian.org/](https://www.debian.org/)
