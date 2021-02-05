

# 안전한 PostgreSQL - Security Hardening Best Practices &Tips



> 참고 문서 : https://www.enterprisedb.com/blog/how-to-secure-postgresql-security-hardening-best-practices-checklist-tips-encryption-authentication-vulnerabilities

본 글은 위의 참고 문서를 기반으로 필요한 내용만 발췌하고 수정 또는 추가하였습니다.  위 문서는 postgres에 대해서 자세하게 다루고 있습니다. 관심 있으신 분들은 참고하세요. 



## Access

어떠한 보안 리뷰에서 처음으로 보는 부분은 어떻게 서버가 연결되고, 접속 되는가이다. 다른 보안 설정과 마찬가지로 시스템 설정을 어떻게 할 것인가를 고려할 때는 '최소 권한 개념(Principle of least privilege)'을 따른다. 즉, 작업 시스템을 구현하는데 필요한 만큼의 접근만 허용하고 더 이상은 허용하지 않는다. 

### Physical access

서버에 물리적으로 접근할 수 있는 사람들이 데이터에 접근하는 것을 막는 것은 매우 어려울 수 있지만, 물리적 혹은 기술적 측면에서 여러가지 조치를 적용할 수 있다. 

다른 무엇보다도 안전한 시설에 서버를 위치시킴으로서 가능한 물리적인 접근을 제한해야 한다. 

이와 관련해서 주요 클라우드 프로바이더가 주장하는 높은 수준의 물리적 보안을 적용한다고 믿는 것 외에는 이와 관련되어 수행할 수 있는 일이 거의 없다. 

### Connecting 

Postgres 서버에 접속하는 방법은 2가지 가 있다. Unix Domain Socket 혹은 TCP/IP Socket

#### Unix Domain Socket

Unix Domain Socket (UDS)는 Postgres 데이터베이스에 연결하기 위한 기본적인 방식이다.  윈도우는 현재 (v.12) 지원하고 있지 않으며, v13 이상에서 지원할 예정이다. 

UDS는 해당 시스템이 있는 시스템에서만 접근 할 수 있어 직접적으로 원격 공격을 받지 않는다. 이것은 파일 시스템에 특수한 파일 형태로 사용한다.

만약 어플리케이션이 데이터베이스와 같은 호스트에서 실행되는 경우 UDS 만을 사용하여 접근을 허용하는 것을 적극적으로 검토해야 한다.

#### TCP/IP Socket

원격 시스템에서 Postgres 서버에 접근해야 하는 경우  TCP/IP 소켓을 사용해야 한다. 

이 방식을 사용하는 경우 일반적인 보안 개념에서 고려하 듯이 시스템에 대한 접근을 시도하는 모든 사용자의 잠재적인 공격 대상 영역을 최소화 하도록 해야 한다.  즉, 네트워크로 접근 가능한 호스트를 제한하는 형태로 네트워크 접근 통제를 적용하는 것이 바람직하다. 기본적으로 Postgres의 소스 코드 빌드 방식으로 설치하는 경우 로컬 호스트 또는 루프백 주소에서만 수신하므로 다른 시스템과의 연결이 차단된다. 그러라 패키지 방식을 이용한 설치의 경우 다른 설정이 되어 있을 수 있어 확인이 필요하다. **postgresql.conf 의 listen_address 파라미터를 사용하여 Postgres 가 필요한 네트워크 주소에서만 연결을 수신할 수 있도록 제한해야 한다.** 

#### Firewall

방화벽은 비인가된 소스로부터 네트워크 포트들에 접근을 방지하기 위한 중요한 도구이다. 또한 많은 사람들이 공격 시도가 성공적으로 완료되기전에 이를 완화시키는데 도움을 줄 수 있도록 사전에 침입 시도를 탐지하기 위한 한 부분으로 사용하는 로깅 기능을 제공한다. 

##### Local machine

대부분의 최근 OS들은 로컬에서 사용할 수 있는 방화벽 기능을 제공하고 있다. 리눅스의 Iptables, 윈도우위 Windos Defender 방화벽 그리고 이외에 써드 파티 제품들도 많다.

항상 그렇듯 Postgres에 대한 접근을 최소화하는 것이 보안 관점에서 목적임으로 어플리케이션 서버의 주소에서 오는 트래픽이 아니라면 port 5432에 도착하는 TCP 트래픽을 거부하도록 규칙을 만드는 것이 일반적인 접근 방법이다. 소스 주소는 ip address 또는 서브넷 목록을 기반으로 적용할 수 있다. 



방화벽에서 핵심은 **방화벽을 사용하여 서버에 대한 접근을 최대한 최소화하는 것**이다. 

##### Cloud provider

대부분의 클라우드 제공자는 가상 인스턴스에서 방화벽을 사용하지 말 것을 권장한다. 사용자가 플랫폼에 내장된 방화벽을 사용할 것을 제안한다. 이렇게 하면 일반적으로 관리가 훨씬 쉬워져 여러 서버에 다시 사용하고 연결할 수 있는 규칙 집합을 만들 수 있으며 웹 및 명령줄 인터페이스와 REST API를 통해 관리 할 수 있기 때문이다.

클라우드의 경우 VPC(Virtual Private Cloud)를 제공하여 가상의 네트워크를 따로 구성할 수 있습니다. 이러한 가상 네트워크 구성은 방화벽과 유사한 기능을 제공할 수 있다 .

#### Transport Encryption

데이터베이스 서버에 대한 트래픽이 네트워크를 통해 흐르는 경우 해당 트래픽을 암호화하는 것이 좋다. Postgres는 한동안 TLS(이전에는 SSL)를 사용하여 Microsoft Secure Channel 또는 Schannel 및 Apple Secure Transport에 대한 지원을 추가하는 작업이 진행되었지만 OpenSSL을 사용하여 전송 보안을 제공한다.

Postgres 내에 암호화 연결을 위해서는 서버가 시작할 때 안전하게 입력할 비밀번호를 보호 하기 위해 최소한 서버 인증서, 키가 필요하다. postgres 12 버전에서는 윈도우 passphrase 기능을 치원하지 않는다. 서버 인증서와 키는 ssl_cert_file 과 ssl_key_files 파라미터를 사용하여 지정한다. 

만약 기존의 CA를 사용 중인 경우는 Postgres에 사용할 수 있다. 

TLS 사용시 이를 안전하게 사용하는 것도 중요하다. TLS 를 사용하기 위해서는 여러가지 파라미터들을 설정해야 하는데 잘 설정 할 수 있도록 주의해야 한다. postgres.conf 구성 파일에서 다음 파라미터들을 확인하고 적절하게 구성해야 한다. 

- ssl_ciphers
- ssl_ecdh_curve
- ssl_dh_params_file
- ssl_min_protocol_version

이 파라미터에 대한 설정들은 시간이 지나면서 지속적으로 바뀌기 때문에 이 문서에서는 다루지 않는다. 다만 안전한 옵션을 계속 사용 중인지 주기적으로 확인하고 필요한 경우에는 업데이트를 진행해야 한다. 



## Authentication

접근 다음으로  고려해야 하는 보안 컴포넌트는 클라이언트의 인증이다. 사용자를 인증하고 pg_diff.conf 구성 파일을 통해 서버에 성공적으로 연결할 수 있는지 여부를 제어하는 방법을 의미한다. 

### pg_hba.conf

pg_hba.conf 파일은 PostgreSQL 클라이언트 인증 설정 파일이다. 일반적으로 Postgres 데이터 디렉터리에 위치 한다. 예) ` /var/lib/postgresql/data/`

pg_hba.conf 파일(전형적으로 Postgres 데이터 디렉터리에 위치함)은 데이터 서버의 접근 규칙과 인증 방법을 정의한다. 이 설정 파일은 데이터베이스에 연결이 수립될 때 이 설정 파일을 순차적으로 매칭이 되는지 확인하고 연결 속성 혹은 어떤 인증 방식을 사용할지를 결정하게 된다.  (일반적으로 방화벽 룰도 맨 위에서부터 룰을 비교하면서 매칭되는지 확인하는 방식을 사용한다.)

이 설정 파일에서는 7가지의 다른 포맷을 사용할 수 있다. (설정 파일에 #주석을 통해서 잘 설명되어 있다.) 그 중 3가지 주요한 부분이 있으며, 나머지는 다른 하나와 동일한 구조를 따른다. 그러나 첫번째 필드에는 다른 연결 유형을 사용한다. 아래는 몇가지 예시이다.

~~~
local my_db my_user scram-sha-256
~~~

위 예에서는 scram-sha-256을 사용하여 my_user 가 로컬(UDS 사용) 연결 방식을 사용하여 my_db에 대한 연결 시도를 허용한다. 

~~~
host	my_db	my_user	172.16.253.47/32	md5
~~~

위의 예에서는 my_user가 my_db 데이터베이스에 172.16.253.47로부터 md5를 이용한 연결 시도에 대해서 허용한다. 

추가로 위의 예에서 /32 표현을 하고 있는데, 주소의 맨 마지막에 붙이는 것은 해당 아이피의 서브넷을 의미한다. 172.16.0.0/16은 앞의 2개 숫자로 시작하는 B클래스의 서브넷 모든 아이피를 가진 호스트들의 접속 허용을 의미한다. 

각 줄의 필드는 항상 연결유형 (Connection type), 데이터 베이스 이름(들), 사용자 이름(들), 클라이언트 네트워크 주소/서브넷, 인증 방식, 추가로 다른 옵션들도 적용 가능하다. 자세한 내용은 [공식문서](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)를 참조할 수 있다. 

다음 절들은 대부분 공통적으로 사용되는 인증 방식들에 대해서 설명한다. 

#### Trust

Trust 인증 방식은 추가 인증 없이 매칭되는 클라이언의 연결을 허용할 수 있으므로 예외적인 경우에만 사용해야 한다. 

이 방식(Trust 인증)은 UDS를 통해서 연결되는 로컬 기기에 개발 혹은 테스트 작업에 유용하며 데이터 보안은 문제가 되지 않는다. 

또한 로그인하는 다른 방법이 없는 경우 서버에서 암호를 재설정하는 데 유용한 매커니즘입니다. 임시로 UDS를 통해서 패스워드를 리셋하기위해 접근  trust 접근을 허용하고 완료 후 해당 기능을 불능화한다.

trust는 극도의 주의를 기울여야 한다. 이것의 사용은 매우 위험할 수 있다. 

#### Peer & Ident

Peer와 ID는 모두 기본 운영 체제에서 사용자를 인증할 수 있는 방법이다. 많은 postgres 패키지는 peer 인증을 사용하도록 구성되어 있습니다. (사용자 아이디 기반의 로그인 방식을 의미하는 것으로 보임)

peer 인증 방식은 로컬 연결에만 사용할 수 있다. peer 사용시 서버는 OS에서 클라이언트의 사용자 이름을 가져와 요청된 데이터베스 사용자 이름과 일치하는지 확인한다. 

ID 인증 방식은 네트워크 연결에만 사용할 수 있다. 사용자 이름을 확인하기 위해 클라이언트에서 실행되는 ID 서버에 의존한다는 점을 제외하고 peer 인증과 유사한 방식으로 작동한다. 

peer 와 ID 인증 방식은 모두 연결 맵을 사용하여 클라이언트에 알려진 사용자 이름과 데이터베이스 서버에 알려진 사용자 이름 간의 허용 가능한 불일치를 처리할 수 있다. 

ID는 클라이언트 신뢰성이 보장되지 않음으로 ID에만 의존해서는 안된다. 

#### md5 vs. SCRAM

많은 해동안 md5는 posgres를 사용하기 위해 선호되던 해싱 매커니즘이었다. 아직까지도 많은 곳에서 사용되고 있으나, 강하게 scram-sha-256 으로 이동하거나 패스워드 인증으로 변경하기를 추천한다. 

 md5와 scram-sha-256 모두 스니핑을 방지하기 위해 challenge-response 매커니즘을 사용하고 해시된 비밀번호를 서버에 저장한다. 그러나 scram-sha-256이 md5에 비해 현재 훨씬 더 안전한 매커니즘이다. (MD5는 사용하지 않는 것을 권장한다.)

패스워드 인증을 지원하는 standalone postgres 서버는 인증 방법으로 scram-sha-256을 사용해야한다. 새로운 배포를 할 때 md5는 사용하지 않아야 한다.

#### LDAP vs. Kerberos

LDAP 및 Kerberos는 SSO 시스템과 통합될 때 기업 환경에서 사용하는 경우가 많다. 이러한 시스템 환경에서는 postgres 서버는 LDAP 디렉토리 혹은 Kerberos 프레임웍을 통해 사용자를 인증하도록 구성된다. 

LDAP 시스템에서 사용자 접근을 제어 할수 있는 여러가지 방법들이 있다. 

Kerberos 인증은 Postgres의 gsapi 인증 방법을 통해 사용할 수 있다. 

LDAP 인증은 작성자가 받은 사용자 피드백에 따라 매우 인기가 있지만, LDAP와 ㄷ라리 사용자으 비밀번호가 Postgres 서버로 전성되지 않으므로 항상 Kerberos 인증을 선호해야한다. 

#### TLS  Certificates

TLS 인증서는 통신 암호화 외에 인증에서 사용할 수 있다. 

인증서 인증은 최상위 인증서(또는 하위 인증서 또는 '중간' 인증서 중 하나)를 신뢰하여 신뢰할 수 있는 클라이언트에만 인증서를 발급하는 방식으로 작동한다. 서버 인증서 및 키도 발급한 상급 기관에서 발급한 인증서 및 키를 보유한 클라이언트는 신뢰된 것으로 간주할 수 있다.

### Addtional Configuration

고려해볼만한 추가적인 옵션이 2개가 있다. 

authentication_timeout 은 postgresql.conf 내에 설정할 수 있는 파라미터이다.  서버가 연결을 종료하기 전에 인증을 완료해야 하는 최대 시간을 설정하는 것이 목적이다. 이것은 불완전한 연결 시도가 무기한으로 연결 슬롯을 차지하지 않도록 보장한다. 

auth_delay 는 postgresql.conf의 shared_preload_library 구성 옵션을 통해 로드할 수 있는 파라미터로 brute force 공격 발생 시 잠시 중지 시킬 수 있는 기능을 제공한다. 

#### Other authentication methods

이외에도 postgres 서버는 더 많은 인기 있는 인증 방식을 지원한다. 다른 인증 방식은 [공식 문서](https://www.postgresql.org/docs/12/client-authentication.html)를 참고할 수 있다. 

## Roles

Postgres  배포를 안전하게 하기 위한 보안 컴포넌트는 특정 사용자들을 위한 데이터 베이스 접근을 제할 수 있는 역할을 생성하고 적용하는 것이다.

### What is a role?

 매우 오래된 postgres 버전 SQL은 사용자 계정을 그룹화하는 방법으로 사용자와 사용자 그룹을 제공했다.  PostgreSQL 8.1 은 SQL 표준 호환 role 시스템으로 대체되었다. 

역할은 다른 역할들의 맴버가 될 수 있거나 이것의 맴버들이 되는 역할을 가질 수 있다. 우리는 다른 역할에 역할을 부여하는 것 처럼 이것을 언급한다. 역할은 데이터베이스 서버에 로그인할 때 사용하는 사용자 계정을 효과적으로 만들 수 있는 여러가지 특성들이 있다.  다른 역할에 역할을 부여하는 예는 다음과 같다. 

~~~sql
GRANT pg_monitor TO nagios;
~~~

위의 예는 nagios 역할을 pg_monitor의 멤버로 만들어, nagios 가 슈퍼유저들을 위해 준비된 확장 기능에 접근을 제공한다. 

### Role attributes

역할에는 다음과 같이 설정할 수 있는 여러가지 고정 특성이 있다. 


![image-20210205001947059](https://user-images.githubusercontent.com/10112510/107052597-37830c00-6811-11eb-9de4-20dc404ae67d.png)

SUPERUSER 플래그가 설정된 역할은 로그인 권한을 제외한 모든 권한 검사를 자동으로 무시한다. 

일반적으로 사용되지 않은 다른 역할의 속성들도 설정할 수 있다. 이러한 설정은 [공식문서](https://www.postgresql.org/docs/12/sql-createrole.html)의 내용을 참조할 수 있다. 

SUPERUSER(CREATEDB와 CREATEROLE 과 같은 잠재적으로 위험한 속성들) 매우 주의하여 부여해야한다. 일반적인 작업에는 SUPERUSER 권한이 있는 역할을 사용하지 말아야 한다. 

### Password complexity

EDB Postgres Advanced Server와는 달리 PostgreSQL에는 <span style=color:red> 기본적으로 암호 복잡성 적용 기능이 포함되어 있지 않는다. </span>모듈 플러그인에 암호 복잡성 검사를 수행하는 데 사용할 수 있는 후크가 포함되어 있지만, 사용자가 미리 해시된 문자열을 사용하여 암호를 변경해도 아무런 영향을 미치지 않는다.

[passwrd check module](https://www.postgresql.org/docs/current/passwordcheck.html)은 소스 트리의 Postgres의 기여 디렉토리에서 찾을 수 있으며 대부분의 패키지 세트에 포함됩니다. 이 모듈은 C 개발 작업이 필요하지만 조직의 특정 요구 사항을 충족하는 보다 복잡한 것을 개발하기 위한 예로 사용될 수 있다.

Postgres에서 암호 복잡성을 적용하는 가장 효과적인 방법은 위에서 설명한 대로 LDAP또는 Kerberos와 같은 외부 ID 서비스를 인증에 사용하는 것이다. 

### Password profile

EDB Postgres Advanced Server는 암호와 함께 사용할 수 있는 암호 프로파일 기능을 제공합니다(비밀번호는 일반 텍스트로 전송되므로 이 기능을 사용하지 않음), md5 및 pg_hba.conf에 구성된 scram-sha-256 인증 방법을 제공합니다. 암호 프로파일은 슈퍼 사용자가 구성하고 하나 이상의 역할에 적용할 수 있다.  

### SET ROLE

[SET ROLE](https://www.postgresql.org/docs/current/sql-set-role.html) sql 커맨드는 현 세션의 사용자 식별자가 다른 사용자의 역할로 변경하기 위해 사용자가 사용한다. 이것은 세션에 대한 권한을 추가하거나 제한하는데 사용할 수 있다. 

SET ROLE은 유닉스 시스템의 sudo su -<user> 를 사용하는 것과 유사하다. 이것은 다른 사용자 처럼 SQL 명령을 실행 시키는 것을 지원하기 위해 필수적이다. 

기본적으로 하나의 role이 다른 role의 맴버일 때 이것은 자동적으로 그 role의 권한을 상속받는다. SET ROLE을 효과적으로 사용하기 위해서, 권한 자동 상속을 방지하는 역할을 생성할 때 NOINHERIT 키워드를 사용해야 하게 하고, 필요 시 SET ROLE을 사용하여 권한을 명시적으로 획득해야 한다. 

이외에 SET ROLE 명령어는 SET SESSION AUTHORIZATION(세션 권한 부여) 명령도 있으며, 이는 슈퍼유저만 사용할 수 있다. SET ROLE 명령어 수행 시 현재 사용자는 변경하지만 기존의 session_user는 변경되지 않지만, SET SESSION AUThORIZATION은 현재 사용자와 session_user 모두 변경한다 부분이 두 명령어 차이점이다. 

잠재적으로  위험한 작업을 수행하는데 필요한 경우에만 사용자가 일시적으로 권한을 상승 시킬 수 있도록 하려면 SET ROLE을 사용하는 것이 좋다. 

#### Monitoring Roles

Postgres는 이전 버전의 Postgres에서만 슈퍼 유저로 제한되었던 기능에 접근할 수 있는 여러 기본 제공 모니터링 역할이 포함되어 있다. 이러한 역할을 사용하면 슈퍼유저 권한을 부여하지 않고도 시스템을 모니터링하는데 사용하는 역할을 부여할 수 있다. 

![image-20210205102722932](https://user-images.githubusercontent.com/10112510/107052573-2e923a80-6811-11eb-92b5-a3971bac39c1.png)

이러한 기능은 최소 권한 유지에 도움이 되는 것으로 모니터링만을 위한 계정에 슈퍼 유저 권한을 부여할 필요가 없어 보안 위협을 줄일 수 있다. 

## Data Access Control

Postgres 배포를 위한 보안 설정의 예로서 데이터 접근 통제와 접근할 수 없는 데이터에 접근하지 못하게 하는 방법을 살펴본다. 



### ACLs

Access Control Lists 나 ACL은 Postgres 내의 컬럼, 뷰, 기능, 테이블 같은 오브젝트에 붙여지는 다소 암호 문자열 같은 것이다. 이것은 각 역할에 부여된  select, insert, execute  같은 실제적으로 권한의 목록과 해당 역할이 있는 경우 다른 역할에 이 권한을 부여할 수 있는 기능과 권한을 부여한 역할의 이름을 나타내는 추가 옵션 플래그들을 포함한다. 

Joe가 작성한 테이블의 ACL 예제는 다음과 같다.

~~~
joe=arwdDxt/joe =r/joe sales_team=arw/joe
~~~

첫번째 부분에서 joe가 테이블을 만들 때 원래 joe에 의해 부여된 모든 사용 가능한 권한(insert, select, update, delete, truncate, reference, triger)을 확인할 수 있다. 

두번째 부분 (`=r/joe`)에서 읽기 접근 권한을 public(모든 사람을 의미)에 부여되었음을 의미하며 세번째 부분(`sales_team=arw/joe`) 는 sales_team 역할에 insert, select, update 권한을 부여 되었다는 것을 의미한다. 

ACL의 권한 플래그는 오브젝트의 타입에 따라 적용할 수 있는 부분이 달라서 자세한 내용은 [권한에 관한 문서](https://www.postgresql.org/docs/current/ddl-priv.html)를 참고하는게 바람직하다.

 일반적으로 내부 형식으로 보여질 수 있는 커맨드 라인 도구 사용을 선호한다면 Postgres 내에서 ACL을 작성하는 방법에 대해서 이해하는게 유용하다. [pgAdmin](https://www.pgadmin.org/) 과 같은 그래픽컬 도구는 후러씬 읽기 쉬운 시각적인 형태로 ACL을 분석하고 표시한다.  

사용자 및 어플리케이션이 작업을 수행하는데 필요한 권한을 잘 이해하고, 각 권한에 따라서 역할들을 최소화하여 접근 제어 리스트를 작성해야 한다. 이를 위해서 필요한 권한만 최소화하고 스키마 소유권을 데이터와 분리하며 그룹 역할을 사용하여 개별 로그인 역할에 대한 권한 관리를 단순화해야한다. 

### GRANT & REVOKE

ACL들은 GRANT와 REVOKE SQL 명령의 사용을 통해서 Postgres 내의 오브젝트에 대해 관리된다. 

오브젝트를 삭제하거나 수정을 위한 권한은 항상 오브젝트의 소유자 와 슈퍼유저들을 위해 준비되는데, 오브젝트 소유권은 Alter SQL 명령을 사용하여 재할당 할 수 있다. 

 기본 권한들은 일부 오브젝트 유형에 대해 ALTER DEFAULT PRIVILEGES를 사용하여 재정의할 수 있다.  이렇게 하면 생성된 새 오브젝트의 역할에 특정 권한이 자동으로 부여되도록 시스템을 구성할 수 있다.  예를 들어 이전 예에서 joe는 아래 명령을 실행하여 sale_team이 새 테이블에 대해 insert, select, update 권한을 부여할 수 있다. (기존 테이블의 경우 수동으로 업데이트 해야 할 수 있음)

~~~sql
ALTER DEFAULT PRIVILEGES 
    GRANT INSERT, SELECT, UPDATE 
    ON TABLES 
    TO sales_team;
~~~

새 오브젝트가 작성될 때 ACL에 필요한 권한이 자동으로 포함되지 않는다고 가정하며, 우리는 GRANT와  REVOKE를 요구되는 ACL을 설정하기 위해 사용할 수 있다. 이전 예제를 계속 진행하기 위해 joe는 다음 SQL 명령을 사용하여 주문 테이블에 대한 sale_team 권한을 부여할 수 있다. 

~~~sql
GRANT INSERT, SELECT, UPDATE ON orders TO sales_team;
~~~

자동으로 부여된 권한을 취소하거나 변화하는 비지니스 요구를 중족하기 위해 이전에 부여된 권한을 해지하려면 REVOKE SQL 명령어를 사용할 수 있다.

~~~sql
REVOKE UPDATE ON orders FROM sales_team;
~~~



> 로그인 사용자를 구성하기 위한 그룹 역할을 생성한 후 GRANT 와 REVOKE 명령을 사용하여 그룹 역할에 작업에 필요한 최소 수준의 권한을 할당한다. 관리 상 효율성을 위해 기본 권한 개념을 사용하되 나중에 적절한 권한보더 더 많이 부여되지 않도록 주의해서 사용해야 한다. 

### RLS (Row Level Security)

RLS는 Postgres에서 사용할 수 있는 기술로, 테이블의 행에 대한 가시성을 특정 역할로 제한하는 정책을 정의 할 수 있다.  RLS 정책을 설정하는 방법을 알아보기 전에 2가지 중요한 주의 사항이 있다.

1. Bypass RLS 특성을 가진 슈퍼유저와 role은 정책을 직접 실행하지 않는 한 테이블 소유자와 같이 항상 행 수준 보안 정책을 무시한다. 
2. 행의 존재는 은닉 채널을 통해서 추론할 수 있다. 예컨대, 사회 보장 번호와 같은 필드는 유니크한 제약이 걸려 있어 사용자가 동일한 값을 가진 다른 행을 삽입할 수 없다. 사용자는 행에 접근 할 수는 없지만 해당 사회 보장 번호를 가진 레코드가 이미 존재한다고 추론할 수는 있다. 

기본적으로 행 수준 보안 (RLS)는 Postgres의 테이블에서 꺼져있다.  이 기능은 다른 정책이 생성되지 않거나 생성될 때까지 모든 데이터에 대한 접근을 금지하는 제한을 활성화 할 수 있는  ALTER TABLE ... ENABLE ROW LEVL SECURITY,  같은 명령어를 통해서 활성화할 수 있다. 

Policy 자체는 이름, 정책이 적용되는 테이블, 이 것이 적용되는 선택적인 role, 허용되는 혹은 매칭되는 row가 식별될 수 있는 방법에 대한 절로 구성된다. 예를 들면  주문을  작성한 Sale Team 맴버에 대한 주문 접근을 제한할 수 있다. 

~~~
CREATE POLICY sales_team_policy ON orders TO sales_team
    USING (sales_person = current_user);
~~~

정책이 적용되는 작업도 지정할 수 있다. 다음 예는 Sales team의 모든 구성원이 주문을 선택할 수 있지만 원래 영업 담당자만 주문을 업데이트 하거나 삭제할 수 있다. 

~~~
CREATE POLICY sales_team_select_policy ON users
    FOR SELECT
    USING (true);
CREATE POLICY sales_team_modify_policy ON users
    USING (sales_person = current_user);
~~~

기본적으로 허용 가능한 정책이 사용되므로 적용되는 정책이 여러개인 경우 boolean OR을 사용하여 결합된다. 

RLS 정책은 설정하기 위해 어느 정도의 노력이 필요할 수 있으며 색인 설계도 이를 고려해야 하지만, 의료 기록 시스템에서처럼 환자의 치료를 직접 담당하는 의료진에 대한 환자 기록 접근을 제한하는 것이 법적 요건일 수 있다.

각 테이블의 특정 행에 대한 접근을 제한하기 위한 법적 및 윤리적 요구사항을 고려하고, 필요한 경우 이러한 요구사항을 만족할 수 있도록 RLS 정책을 설계 및 구현한다. 민감한 데이터를 제약 조건에 사용하지 않도록 함으로서 컨버트 채널을 최소화하도록 노력해야 한다. 

 

#### Views

Views는 일반적으로 실행되는 쿼리를 테이블처럼 쿼리할 수 있는 개체로 캡슐화하는 데 분명히 유용하지만, 역할이 기본 테이블에서 선택할 수 있는 기능이 없고 view서 데이터에 액세스해야 하므로 데이터에 대한 무단 액세스를 방지하는 데도 유용할 수 있습니다.



#### Security barriers

열(column)에 대한 접근을 제한하기 위해 VIEW를 사용하는 것은 일반적인 일이지만, 사람들은 종종 보기를 사용하여 특정 행(row)에 대한 엑세스를 제한하기도 한다. 이런 사용 방식도 분영한 가치가 있으나 한가지 특히 주의해야할 부분이 있다. Postgres 최적화를 속여 숨겨진 데이터를 유출할 가능성이 있다. 

이것은 실제적인 버그가 아니다. 시스템이 의도된데로 작동하는 것이다. 이와 관련된 내용은 이 [블로그 포스트](http://rhaas.blogspot.com/2012/03/security-barrier-views.html)에 잘 정리되어 있다. 

위의 문제를 해결하기 위해서 기본적으로 view가 생성될 때 통과되는 하나의 view를 첫번째로 항상 사전에 실행되도록 하는 옵션을 사용함으로서 숨겨진 rows를 함수가 보지 못하도록 할 수 있다. 

보안 장벽과 관련된 것은 기능에 대한 Leakproof 매개 변수입니다. 이것은 기능을 생성할 때 오직 슈퍼유저들에 의해 사용될 수 있으며, 함수가 의도한 리턴 값 이외에 정보를 유출하지 않음을 확정해주는 역할을 한다. 이를 통해 Postgres는 기능이 어떠한 정보도 유출하지 않는다는 것을 알 수 있도록 보안 장벽 View와 함게 사용되는 쿼리를 보다 효율적으로 최적화 할 수 있다. 

View를 사용하여 행을 숨길 때 데이터 유출을 방지하기 위해 Security barrier로 표시되도록 주의해야 한다. RLS 가 특정 행에 대한 접근을 제한하기 위한 더 나은 솔루션인지 여부를 고려해야 한다.



#### Security Definer Functions

기본적으로 Postgres내의 함수와 프로시저를 우리는 Security invoker 기능이라고 부른다. 이것은 함수 와 프로시저들은 이를 호출한 역할의 권한과 함께 실행되는 것을 의미한다. 

함수를 만들 때 Scurity Definer 옵션을 전달하면 함수가 호출될 때마다 호출한 role 대신 소유자의 권한으로 실행된다. 이것은 유닉스 시스템의 ACL 의 setuid 기능과  유사한 기능이다. 

이 기능은 다양한 상황에서 유용하게 사용될 수 있다. 

Security Definer 기능을 사용하여 해당 작업을 직접 수행할 수 없는 역할에 특정 기능을 제공하는 것을 고려해야 한다. 그러한 기능이 잘못 사용될 수 있는 가능한 영향과 방법을 고려하여 의도한 작업만 수행하도록 제한되는지 주의해야 한다. 

#### Data redaction

데이터의 수정은 표시되는 값을 동적으로 변경하여 사용자에게 중요한 정보의 특정 부분을 숨기는 기술이다. 이 작업은 위에서 설명한 대로 Postgres의 보기에서 어느정도 수행할 수 있지만 EDB Postgres Advanced Server에는 기본 데이터 수정 기능이 포함되어 있다. 



## Encryption

Postgres에 대한 end-to-end 보안을 실행할 때 고려해야 할 마지막 컴포넌트는 중요한 데이터를 암호화하는 것이다. 암호화를 통해 추가 보안을 제공하도록 구성할 수 있는 여러가지 방법과 extentions들이 있다. 

### pgcrypto

[pgcrypto](https://www.postgresql.org/docs/current/pgcrypto.html)는바이너리 분포 및 소스 트리 내에 기여 모듈로 포함된 EPAS 와 Postres 표준 확장프로그램이다. 데이터베이스 설계에서 논리의 일부로 활용할 수 있는 암호화 및 해시에 SQL 기능을 제공하는 것이 목적이다. 

#### Installation

Postgre의 대부분의 바이너리 배포는 먼저 contrib module이 서버에 설치되어 있는지 확인하여 pgcrypto를 설치할 수 있다.  리눅스 배포만에서의 패키지 이름은  postgresql12-contrib 이다. 

패키지가 서버에 설치가 된 경우, 슈퍼유저 계정으로 데이터베이스에서 단순하게 CREATE EXTENSION 명령어를 실행할 수 있다. 

~~~sql
CREATE EXTENSION pgcrypto;
~~~

규제 및 유사한 요구 사항을 충족하기 위해 개별 데이터를 해시하거나 암호화 할 수 있는 기능이 필요한 경우 데이터베이스의 pgcrypto 확장 기능을 사용해 볼 수 있다. 

#### Hashing

해싱(Hashing)은 일반적으로 고정된 길이의 값을 출력해주는 암호화 방식으로 안전하 데이터 표혀을 생성하는 방법이다. 

해싱은 확인이 필요하지만 반환할 수 없는 암호 및 기타 중요한 정보를 저장하는 데 가장 일반적으로 사용된다. 

예를 들면, pgcypto를 사용하여 사용자가 나중에 사용할 암호를 해시할 수 있다.

~~~sql
INSERT INTO users 
    (username, email, password)
VALUES
    ('pgsnake', 'dave.page@enterprisedb.com', crypt('new password', gen_salt('md5')));
~~~

이 암호를 검증하기 위해서, 테이블로부터 사용자 기록 조회를 위해 select를 사용할 수 있다. 

~~~sql
SELECT
    *
FROM
    users
WHERE
   username = 'pgsnake' AND 
   password = crypt('entered password', password)
~~~

레코드가 반환되면 암호하 올바르게 입력되었다는 것을 확인할 수 있고, 그렇지 않다면 암호가 올바르지 않다고 판단한다. 

#### Encryption

pgcrypto는 또한 검색해야 하지만 안전한 형태로 저장되어야 하는 정보를 저장할 때 유용한 데이터 암호화 기능을 제공한다. 

Symmetric key encryption는 가장 사용하기 쉽다. 예들 들어, 내부 함수 호출이 데이터를 암호화하고 외부 호출이 데이터를 해독하는 간단한 SQL 명령으로 표시된 것 처럼 데이터의 암호화 및 복호화를 시연할 수 있다. 

~~~sql
SELECT pgp_sym_decrypt(
    pgp_sym_encrypt('Hi There', 'password'), 
    'password');
~~~

암호화 함수에 의해 반환되고 복호화 함수에 전달된 암호 텍스트는 바이트 형식이다. 



Postgres 서버는 이외에 Key management, File system and full disk encryption 등의 기능을 제공한다. 이 기능들은 이글의 참고 문서에서 확인 할 수 있다.	 
