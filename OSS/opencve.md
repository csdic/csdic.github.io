# Opencve 리뷰 

> 대상 프로그램 : https://github.com/opencve/opencve
>
> 내부 설치 및 데모 보기 : http://10.106.238.76:8000/
> (미설정으로 메일 전송 기능은 제공하지 않음)



## 기능 검토

- 주기적으로 실행하면서 CVE의 업데이트 된 내용을 업데이트 진행
- CVE(Common Vulnerablities and Exposure),  CPE(Common Platform Enermation), CWE 목록 제공 
  - 각 항목 별 세부 정보 및 관련 링크 제공
- 각 목록에 대한 검색 기능 제공 





## 설치하기 

### PostgreSQL 준비하기

opencves는 CVE 데이터를 저장하기 위해 내부적으로 postgresql을 사용한다. 이 서버를 설치하기 위해서 내가 사용하는 서버의 설정 상태를 변경하기 싫어서 나는 Docker 이미지 형식으로 postgresql을 설치하고 사용한다. 

#### Docker 기반 PostgreSQL 사용시 주의 사항 

Docker 이미지를 사용할 경우 데이터 베이스 저장 파일을 그대로 docker 컨테이너에 저장할 경우, 컨테이너가 삭제되면 기존에 저장된 데이터는 사라진다. 따라서 docker volume를 사용하거나 로컬 컴퓨터 파일을 마운트하여 docker 이미지 내부에서 생성되는 데이터를 저장하도록 설정하는 과정이 필요하다.



~~~bash
#데이터 저장할 볼륨 생성
$ docker volume create pgdata 

#postsql 도커 실행 - 실행 시 volume 연결 옵션 사용
$ docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD=1q2w3e4r -d -v pgdata:/var/lib/postgresql/data postgres
~~~



#### PostgreSQL 설정 

다음과 같이 docker 이미지에 쉘 접속하여 postgresql 쉘을 이용하여 새로운 사용자 및 데이터 베이스를 생성한다. 

~~~
$ docker exec -it postgres /bin/bash #컨테이너이름 대신 ID 입력 가능
root@bf4da76e0f3c:/# psql -U postgres
psql (13.1 (Debian 13.1-1.pgdg100+1))
Type "help" for help.

postgres=# CREATE USER opencve PASSWORD '1234' SUPERUSER;
CREATE ROLE
postgres=# CREATE DATABASE opencve OWNER opencve;
CREATE DATABASE
postgres=# 
~~~



### opencve 설치 

본 절은 [opencve 공식 github 사이트](https://github.com/opencve/opencve) 매뉴얼을 참고하여 작성하였다. 

단순하게 아래 pip 명령어를 이용하여 설치한다. 

~~~
$ pip install opencve
~~~

위의 명령어를 실행하여 opencve와 의존성 있는 패키지를 모두 설치 한 후 다음 명령어를 이용하여 설정 파일을 생성할 수 있다. 

~~~
$ opencve init
[*] Configuration created in /Users/ncrocfer/opencve/opencve.cfg
~~~

위의 명령 수행 시 일반적으로 현재 실행 계정의 홈 디렉터리의 opencve 디렉터리 하위에 opencve.cfg 파일을 생성하게 된다. 



이후에 opencve.cfg 파일에 postgresql 의 데이터베이스 이름 및 계정 정보를 저장한다. 

~~~
#opencve.cfg
...
...
postgres : 
...

~~~



postgresql 계정 정보를 입력한 후에 아래 명령을 수행하여 데이터베이스 내에 테이블을 생성한다. 

~~~bash
$ opencv upgrade-db
~~~

테이블이 생성된 이후에는 cve 데이터를 웹으로 다운로드 받아 내부 데이터베이스에 저장하기 위해 아래 명령어를 수행한다. 아래 명령어는 수행 시간이 많이 걸리기 때문에 기다려야 한다. 

~~~
$ opencv import-data
...
######################
Importing CVE for 2016
######################
[*] Downloading https://nvd.nist.gov/feeds/json/cve/1.1/nvdcve-1.1-2016.json.gz... (done in 2.439s).
[*] Parsing JSON elements... (done in 1.518s).
[*] Creating model objects... (done in 2.374s).
[*] Inserting CVE... (done in 15.592s).
[*] 10352 CVE imported.
######################
Importing CVE for 2017
...
~~~



CVE 와 서버 내 데이터 베이스를 동기화 하기 위해서 opencve는 주기적으로 Celery task를 실행한다. 이를 스케줄링하기 위해서는 opencve.cfg의 아래 항목을 설정해야 한다. 

~~~
$ opencve celery worker -l INFO

 -------------- celery@dev-ossguidelv4.3.0 (rhubarb)
---- **** ----- 
--- * ***  * -- Linux-3.x86_64-with-centos-7.9.2009-Core 2021-02-02 16:44:55
-- * - **** --- 
- ** ---------- [config]
- ** ---------- .> app:         opencve:0x7f0ad6208eb8
- ** ---------- .> transport:   redis://127.0.0.1:6379/0
- ** ---------- .> results:     redis://127.0.0.1:6379/1
- *** --- * --- .> concurrency: 4 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** ----- 
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery
$ opencve celery beat -l INFO

celery beat v4.3.0 (rhubarb) is starting.
__    -    ... __   -        _
LocalTime -> 2021-02-02 16:46:05
Configuration ->
    . broker -> redis://127.0.0.1:6379/0
    . loader -> celery.loaders.app.AppLoader
    . scheduler -> celery.beat.PersistentScheduler
    . db -> celerybeat-schedule
    . logfile -> [stderr]@%INFO
    . maxinterval -> 5.00 minutes (300s)
[2021-02-02 16:46:05,221: INFO/MainProcess] beat: Starting...
~~~

- celery task를 실행하려면 redis 서버를 설치하고 실행해야 한다. 

  ~~~
  #CentOS를 사용하는 경우 EPEL(Extra Packages for Enterprise Linux) Repository가 필요함
  $ sudo yum install epel-release
  $ sudo yum update
  $ sudo yum install redis
  
  $ sudo systemctl start redis
  $ sudo systemctl enable redis 
  
  #redis 설치 확인
  $ redis-cli ping
  PONG
  ~~~

  

### 계정 생성

~~~bash
$ opencve create-user john john.doe@example.com --admin
Password:
Repeat for cofirmation:
[*] User john created.
~~~



### 웹 서버 실행하기 

아래 명령어를 실행하여 opencve 웹 서비스를 실행할 수 있다. -> localhost:8000 

~~~
#localhost에서만 접속 할 경우 
$ opencve webserver

#외부 IP에서 연결 가능
$ opencve webserver -b xx.xx.xx.xx:8000
~~~

