# kakaopay-devops
> 2019 카카오페이 공채 경력 DevOps 과제

Docker를 사용하여 spring-boot-sample-web-ui 어플리케이션을 빌드/배포 환경 구성하

## Getting Started

### Prerequisites
- Intellij
- Spring Boot
- JDK version : 1.8.x
- Docker version : 18.09.2
- docker-compose version : 1.23.2


### How to build and deploy using Docker


#### 1) create directory and template file
```bash
# create host log directory
mkdir -p /tmp/log/app
mkdir -p /tmp/log/nginx

# create host nginx directory
mkdir -p /tmp/nginx/conf.d
mkdir -p /tmp/nginx/templates
cp nginx.tmpl /tmp/nginx/templates/nginx.tmpl
```
#### 2) build step

```groovy
# setting app version in build.gradle file
bootJar {
    baseName = 'sample-web-ui'
    version = '1.0.0'
}
```
```bash
# run gradlew
./gradlew clean build docker
```
```bash
# check docker image
docker images | grep lifeot/sample-web-ui

lifeot/sample-web-ui   1.0.0               1950dbdf2873        2 hours ago         126MB
lifeot/sample-web-ui   latest              1950dbdf2873        2 hours ago         126MB

```
#### 3) deploy step
``` bash
# run docker containers (proxy & app)
docker-compose build --no-cache
docker-compose run -d
docker-compose ps

             Name                            Command                  State                Ports
---------------------------------------------------------------------------------------------------------
proxy                             nginx -g daemon off;             Up             0.0.0.0:80->80/tcp
spring-boot-sample-web-ui_app_1   java -cp app:app/lib/* sam ...   Up (healthy)   0.0.0.0:32850->8080/tcp

# generate nginx reverse configuration file and reload nginx(Using docker-gen)
docker run -d --name proxy-gen --volumes-from proxy \
   -v /var/run/docker.sock:/tmp/docker.sock:ro \
   -v /tmp/nginx/templates:/etc/docker-gen/templates \
   -t jwilder/docker-gen -notify-sighup proxy -watch -only-exposed /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf
```

#### 4) sacle in/out
```bash
# scale out app container
docker-compose scale app=3

Starting spring-boot-sample-web-ui_app_1 ... done
Creating spring-boot-sample-web-ui_app_2 ... done
Creating spring-boot-sample-web-ui_app_3 ... done

docker-compose scale app=1

Stopping and removing spring-boot-sample-web-ui_app_2 ... done
Stopping and removing spring-boot-sample-web-ui_app_3 ... done
````


### TODO
- [ ] Blue/Green Deployment 구성
- [ ] 전체실행/전체중단/그린배포/블루전환/롤링업데이트 배포스크립트 생성
- [ ] 데이터 영속성을 위해 데이터베이서 컨테이너 생성 후 연동 구성