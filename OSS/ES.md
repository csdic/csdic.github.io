# ElasticSearch Security

{% include list.liquid all=true %}



Elasticsearch(ES) 는 여러가지 라이센스를 포함한 버전을 가지고 있다. 아파치 라이센서를 가진 OSS 버전, basic 버전 등 다양하다. basic 버전은 x-pack의 보안 기능을 무료로 사용할 수 있으나, 상용으로 사용할 때는 제한을 받을 수 있다. 

본 문서는 Elasticsearch에 보안 기능을 추가할 수 있는 오픈 소스 플러그인인 Search Guard에 대해서 설명하고 이를 이용하여 ES의 보안을 강화하는 방법에 대해서 다룬다. 



## Search Guard Quickstart

Search Guard 는 ES(Elasticsearch)와  Elastic Stack 전체를 위한 엔터프라이즈 보안 및 알림 제품 군이다. 이 플러그인은 다음과 같은 기능을 제공한다. 

- TLS 암호화
- Elastic indices에 대한 RBAC
- Document, Field-level 보안 통제들 
- 감사 로깅 및 알림 

이 도구는 Kibana, Logstash, Beats를 포함한 ES내에 저장된 데이터에 대한 접근을 관찰하고 안전하게 하고, 암호화하는데 필요한 모든 도구들을 제공한다. 

Search Guard는 다음과 같은 3가지 버전이 있다. 

- Free Community Edition
- Enterprise Edition
- Compliance Edition for covering regulations like PCI, GDPR, HIPAA or SOX

### TLS encyrption for ES

Search Guard는 ES 클러스터 내에 모든 데이터의 흐름(전송 계층, REST) 암호화 한다. 이는 다음을 보장할  수 있다. 

-  어떠한 데이터도 도청할 수 없다. 
- 너의 데이터를 변조할 수 없다. 
- 오직 신뢰할 수 있는 노드들만 ES 클러스터에 가입할 수 있다. 



### ES authentication and authorization

Search Guard는 아래와 같이 인증과 권한 부여에 관한 모든 주요 산업 표준을 지원한다. 

- LDAP and Active Directory
- JSON Web token
- TLS client certificates
- Proxy authentication
- kerberos
- OpenID Connect
- SAML

외부의 인증 도구가 필요하지 않거나 원하지 않을 경우 내부의 사용자 데이터베이스를 사용할 수 있다. 

### ES security controls

Search Guard는 RBAC를 ES 클러스트와 indices에 추가할 수 있다. Search Guard 역할들은 사용자가 주어진 ES index에 대해 허용되는 사용자의 활동을 정의하고 통제할 수 있다. 역할들은 클러스트 재시작 혹은 노드의 중단 없이 정의되고 할당 될 수 있다. 

접근 권한들을 정의하기 위해서 READ, WRITE, DELETE 처럼 활동 그룹들을 미리 정의할 수 있다. 또한 활동 그룹을 정의하여 사용함으로써 더 구체적인 접근 통제를 수행할 수 있다. 

Index 이름을 와일드 카드, 정규 표현식, 날짜 수학 표현, 변수등을 이용하여 동적으로 역할을 정의할 수 있다. 



ES 데이터에 Document , Field 수준의 접근 지원

Search Guard 는 ES indices 내에 문서 및 필드에 접근을 통제할 수 있다.  Search Guard 역할을 다음과 같이 정의할 수 있다. 

- index 내에 사용자가 접근 할 수 있는 문서가 무엇인지
- 사용자가 접근할 수 있는 필드 정의
- 익명으로 접근 가능한 필드 

이러한 사항들은 런 탕임에 정의가 가능하다. 기존에 존재하는 indices와 데이터 에 대한 통제 적용 뿐만 아니라 ingest time에도 이러한 사항에 대해 결정할 수 있다. 



### Audit Logging

Search Guard ES 클러스터 내에 모든 데이터 흐름을 모니터하고 추적하며, 여러 레벨들의 감사 데이터를 생성할 수 있다. 이러한 감사 데이터는다음을 포함한다. 

- 로그인 시도 실패, 권한 침해, 해더의 도용, 유효하지 않은 TLS 인증과 같은 보안 관련 이벤트들 
- 성공적으로 실행된 쿼리들
- 문서에 읽기 접근 
- JSON 패치 포맷 변경을 포함한 문서에 쓰기 접근

이러한 Search Guard의 감사 로깅은 ES 가 GDPR, PCI, SOX or HIPAA와 같은 규제 지침을 준수하는데 특별히 도움을 줄 것이다. 



### Alerting - Anomaly detection for your Elasticsearch data

Search Guard는 ES 클러스터의 데이터를 주기적으로 검사하여 이상 징후 를 확인하는 알림 모듈과 함께 제공된다. 알림은 Email, PagerDuty, Slack, JIRA 혹은 다른 end 단의 webhook등 다양한 채널들을 통해 공지 된다. 

ES Alerting 모듈은 완전히 새로운 단계적 에스컬레이션 모델을 제공하므로 다음에 따라 다른 채널에서 알림을 보내도록 선택할 수 있다.  Search Guard는 비정상 상황이 더이상 탐지 되지 않고 모든게 정상으로 돌아오면 알린다. 





### 참조 문서

https://docs.search-guard.com/latest/security-for-elasticsearch