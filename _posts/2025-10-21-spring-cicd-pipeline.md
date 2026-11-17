---
title: "[DevOps] 스프링 CI/CD 파이프라인 직접 구축"
date: 2025-10-21 03:52:33 +0900
categories: [DevOps]
tags: []
render_with_liquid: false
image:
  path: /assets/img/posts/spring-cicd-pipeline/cover.png
  alt: 스프링 CI/CD 파이프라인 직접 구축
---
### workflow 파일 생성
루트 디렉토리 기준으로 .github/workflows/ 에 yml 파일을 생성한다.
여기서는 main 브랜치에 대한 ci/cd 를 구축할 예정이다.

### workflow 파일 작성

1. 트리거
워크플로우는 특정 이벤트가 발생하였을 때 트리거가 된다. 일반적으로는 push 또는 PR 이 발생했을 경우에 트리거 되도록 설정이 가능하다.

``` yml
on:
  push:
    branches:
      - main
```

main 브랜치에 push 이벤트가 발생하면 해당 워크플로우가 실행된다.

{: start="2"}
2. 작업(jobs)
실행 할 여러 개의 작업을 작성한다. 각 작업은 독립적으로 실행된다. 깃허브에서 제공하는 VM 또는 컨테이너에서 실행되고, ubuntu, linux 등 다양한 실행환경을 제공한다.

- 스텝(steps): jobs 는 여러 개의 스텝으로 구성된다. 각 단계는 특정 작업을 실행하고, 명령어를 실행하거나 외부 라이브러리를 사용할 수 있다.

jobs 안에서 우리는 build 단계와 deploy 단계를 작성할 것이다. 


### Build 작성

1. 레포지토리 체크아웃

``` yml
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: checkout repository
        uses: actions/checkout@v3
```
main 브랜치에 체크아웃을 한다. 

---

{: start="2"}
2. JDK 설치

``` yml
      - name: Set up JDK21
        uses: actions/setup-java@v4
        with:
          java-version: 21
          distribution: 'temurin'
          cache: 'gradle'
```
깃허브 Action 머신에 JDK 를 설치한다. 
- java-version: 설치할 JDK 의 버전.
- distribution: 빌드 기관 설정(필수)  temurin이 가장 널리 사용됨.
- cache: 프로젝트가 사용하는 의존성들을 캐싱.

---

{: start="3"}
3. 스프링 프로젝트 빌드

``` yml
      - name: Build with gradle
        working-directory: ./backend
        run: ./gradlew build

```
gradle 을 사용해서 빌드를 진행한다.
github directory -> root directory 구조로 되어있다. root 는 깃허브 레포지토리의 첫 화면이라고 생각하면 된다.
그래서 working-directory 를 작성하여 build 가 있는 디렉토리를 가리키도록 하였다.

---

{: start="4"}
4. 도커 허브 로그인

``` yml
      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

---

{: start="5"}
5. 도커 이미지 빌드 및 푸시

``` yml
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and Push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./verify-student-api/
          file: ./verify-student-api/Dockerfile
          push: true
          tags: |
            ${{ secrets.DOCKER_HUB_USERNAME }}/didacto-app-dev:latest
            ${{ secrets.DOCKER_HUB_USERNAME }}/didacto-app-dev:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha, mode=max
```
Buildx 의 경우 기본 도커 빌드 도구를 Buildx 라는 엔진으로 업그레이드 하고 활성화 하라는 의미이다. 다음 단계인 cache-to 에서 캐시를 외부로 저장하는 기능을 지원한다.
<span style= "color: red">사용하지 않으면 다음 단계의 cache 부분에서 에러가 날 수 있다.</span>
- uses: 도커 image 를 빌드하고 푸시하는 전문 액션(Action) 을 사용한다고 명시.
- context: Dockerfile 이 존재하는 디렉토리를 가리키게 한다. Dockerfile 안의 `COPY` 명령어가 이 디렉토리를 기준으로 파일을 찾는다.
- file: Dockerfile 자체를 가리키게 한다.
- push: 이미지를 빌드하고 도커 허브에 푸시할지 결정.
- tags: 빌드된 도커 이미지에 붙일 태그를 지정. github.sha 는 Github Actions 가 제공하는 변수
- cache-from, to: 캐싱

<br />

### Deploy 작성
앞서 설명하였듯이 각 작업은 독립적으로 실행된다. 그러므로 실행환경을 한 번 더 명시해주어야 한다.

---

1. 실행환경

``` yml
  deploy:
    runs-on: ubuntu-latest
    needs: build
```
ubuntu 최신버전에서 실행하고, build 가 성공적으로 완료되어야 실행이 된다.

---

{: start="2"}
2. step 작성

``` yml
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
```
build 단계에서와 같이 체크아웃 해준다.

---

{: start="3"}
3. flyio CLI 설치

``` yml
      - name: set up flyio CLI
        uses: superfly/flyctl-actions/setup-flyctl@master
```
배포에 빌표한 flyio 를 먼저 설치해준다.

---

{: start="4"}
4. 배포

```yml
      - name: Deploy to Fly.io
        working-directory: ./verify-student-api
        run: flyctl deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```
working-directory 를 다시 명시해주고, 실행할 명령어를 run 에 작성한다.
환경변수로 Fly.io 의 토큰을 추가해준다. 


---

### 완성된 workflow 파일

``` yml

name: Deploy to Fly.io


on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: checkout repository
        uses: actions/checkout@v3

      - name: Set up JDK21
        uses: actions/setup-java@v4
        with:
          java-version: 21
          distribution: 'temurin'
          cache: 'gradle'

      - name: Build with gradle
        working-directory: ./verify-student-api
        run: ./gradlew build

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and Push Docker image
        uses: docker/build-push-action@v5
        with:
          context: ./verify-student-api/
          file: ./verify-student-api/Dockerfile
          push: true
          tags: |
            ${{ secrets.DOCKER_HUB_USERNAME }}/didacto-app-dev:latest
            ${{ secrets.DOCKER_HUB_USERNAME }}/didacto-app-dev:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha, mode=max
  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: set up flyio CLI
        uses: superfly/flyctl-actions/setup-flyctl@master

      - name: Deploy to Fly.io
        working-directory: ./verify-student-api
        run: flyctl deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}


```