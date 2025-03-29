# RDS Proxy

## Proxy?

프록시(Proxy)는 클라이언트(사용자)와 서버(목적지) 사이에서 중계 역할을 하는 서버 또는 소프트웨어입니다.  
즉, 클라이언트가 직접 서버와 통신하는 것이 아니라, 프록시 서버를 거쳐서 요청을 보내고 응답을 받는 방식입니다.  

<br>

프록시의 주요 기능은 아래와 같습니다.

1️. 네트워크 보안 강화  
    - 클라이언트와 서버 간의 직접적인 연결을 차단하여 보안을 높임  
    - DDoS(분산 서비스 거부) 공격 방어 및 방화벽 역할 수행 가능  

2️. IP 주소 숨기기 및 우회  
    - 사용자의 실제 IP 주소를 감춰서 익명성을 제공  
    - 특정 지역에서 차단된 콘텐츠 접근 가능 (예: 유튜브, 넷플릭스 우회)  

3️. 캐싱(Caching)  
    - 자주 요청되는 데이터를 미리 저장하여 성능을 향상  
    - CDN(Content Delivery Network)에서 웹사이트 이미지를 캐싱하여 로딩 속도를 빠르게 함  

4️. 트래픽 분산 및 부하 감소  
    - 많은 요청을 처리할 때, 서버의 부하를 줄이기 위해 프록시가 일부 요청을 캐싱하거나 로드 밸런싱 수행  

<br>

프록시의 종류는 크게 네가지입니다.  

1. 정방향 프록시 (Forward Proxy)  
    - 사용자의 요청을 대신 서버로 전달하는 방식  
    - 주로 개인 사용자나 기업에서 보안 및 우회 접속을 위해 사용됨  
    - 예: 회사 내부에서 특정 웹사이트 차단 시, 직원들은 프록시 서버를 통해 인터넷에 접속 가능  

2. 역방향 프록시 (Reverse Proxy)  
    - 서버 앞에 위치하여 클라이언트의 요청을 대신 처리  
    - 부하 분산(로드 밸런싱), 캐싱, 보안 강화 역할 수행  
    - 예: Nginx, Apache, AWS RDS Proxy 등  

3. 오픈 프록시 (Open Proxy)  
    - 누구나 사용 가능한 공개 프록시  
    - 보안 위험이 크며, 해커들이 악용할 가능성이 높음  

4. 트랜스페어런트 프록시 (Transparent Proxy)  
    - 사용자가 인식하지 못한 채 자동으로 적용되는 프록시  
    - 주로 기업이나 ISP(인터넷 서비스 제공업체)에서 사용  

<br>
<br>

## Amazon RDS Proxy

Amazon RDS Proxy는 **Amazon RDS(Aurora 포함)**를 위한 관리형 데이터베이스 프록시로, 데이터베이스 연결을 최적화하여 애플리케이션의 성능과 가용성을 높여줍니다.  

RDS Proxy는 **데이터베이스 연결 풀링(connection pooling)**을 제공하여, 많은 수의 짧은 연결이 발생하는 애플리케이션에서도 데이터베이스 부하를 줄이고 효율적으로 관리할 수 있도록 합니다.

<br>
<br>

## RDS Proxy 주요 기능

- Connection Pooling (연결 풀링)
  - 데이터베이스 연결을 재사용하여 신규 연결을 설정하는 비용을 절감합니다.  
  - 다수의 클라이언트가 요청을 보낼 때, 기존 연결을 재활용하여 성능을 향상시킵니다.  
  - 애플리케이션의 데이터베이스 연결 수를 최소화하면서도 더 많은 요청을 처리할 수 있도록 합니다.  

<br>

- Failover 속도 개선  
  - RDS/Aurora가 **장애 조치(failover)**를 수행할 때, RDS Proxy는 애플리케이션의 연결을 유지하면서 빠르게 새로운 데이터베이스 인스턴스로 라우팅합니다.  
  - Aurora의 경우, 일반적인 장애 조치 시간이 30초~1분이지만, RDS Proxy를 사용하면 수 초 이내로 줄어듭니다.

<br>

- IAM 인증 및 보안 강화  
  - AWS IAM 인증을 지원하여, 애플리케이션이 데이터베이스 자격 증명 없이 안전하게 인증할 수 있습니다.  
  - AWS Secrets Manager와 통합되어, 데이터베이스 접속 정보를 안전하게 관리할 수 있습니다.  

<br>

- 읽기 및 쓰기 요청 최적화  
  - Aurora 클러스터에서 **읽기 전용 노드(read replica)**를 활용하여, 읽기 요청을 자동으로 라우팅할 수 있습니다.
  - 기본적으로 쓰기 요청은 마스터 노드로 전달되며, 읽기 요청을 분산할 수 있는 기능을 제공합니다.  

<br>

- 자동 확장 및 성능 최적화  
  - 다중 데이터베이스 연결을 효과적으로 관리하여, 연결이 급격히 증가해도 성능 저하 없이 대응할 수 있습니다.  
  - 연결을 유지하는 동안 비활성 연결을 자동으로 종료하여 불필요한 리소스 사용을 줄입니다.

<br>
<br>

## RDS Proxy 설정 방법

- RDS Proxy 활성화
    1. AWS 콘솔에서 RDS > Proxies > "Create Proxy" 선택
    2. Proxy 이름, VPC 및 서브넷 설정
    3. 연결할 RDS 인스턴스 선택
    4. 인증 옵션 (IAM 인증 또는 RDS 자격 증명) 설정

<br>

- 애플리케이션에서 RDS Proxy 사용
  - 기존의 RDS 엔드포인트 대신 RDS Proxy 엔드포인트를 사용하여 연결
  - (예시) pg-pool을 사용한 연결 풀링  
    : RDS Proxy는 내부적으로 **연결 풀링(Connection Pooling)**을 제공하지만, 애플리케이션에서도 풀링을 활용하면 더욱 최적화할 수 있습니다.  
  
  ```typescript
    import { Pool } from 'pg';

    // PostgreSQL 연결 풀 생성
    const pool = new Pool({
    host: 'my-rds-proxy.proxy-xxxxxx.region.rds.amazonaws.com',
    port: 5432,
    user: 'admin',
    password: 'mypassword',
    database: 'mydatabase',
    max: 10, // 최대 동시 연결 수
    idleTimeoutMillis: 30000, // 30초 동안 유휴 상태이면 연결 해제
    connectionTimeoutMillis: 5000, // 연결 제한 시간 5초
    });

    (async () => {
    try {
        const client = await pool.connect();
        console.log('✅ Connected to PostgreSQL via RDS Proxy');

        const res = await client.query('SELECT NOW() AS current_time');
        console.log('🕒 Current Time:', res.rows[0]);

        client.release(); // 풀에 연결 반환
    } catch (err) {
        console.error('❌ Database error:', err);
    }
    })();
  ```

- IAM 인증 방식 사용 (선택)
  - AWS IAM 인증을 사용할 경우, 데이터베이스의 패스워드 대신 IAM 인증 토큰을 생성해야 합니다.
  - IAM 인증을 사용하면 보안이 강화됩니다.

  1. IAM 인증 토큰 생성  

    ```bash
    aws rds generate-db-auth-token \
    --hostname my-rds-proxy.proxy-xxxxxx.region.rds.amazonaws.com \
    --port 5432 \
    --region ap-northeast-2 \
    --username admin
    ```

  2. Node.js 코드에서 IAM 인증 적용  
    : 위 예시에 적용하면 아래와 같습니다.

    ```typescript
    // AWS CLI로 IAM 인증 토큰 생성
    const token = execSync(`aws rds generate-db-auth-token --hostname ${host} --port 5432 --region ap-northeast-2 --username ${username}`)
    .toString().trim();

    const pool = new Pool({
    host: 'my-rds-proxy.proxy-xxxxxx.region.rds.amazonaws.com',
    port: 5432,
    user: 'admin',
    password: token, // IAM 인증 토큰 사용
    database: 'mydatabase',
    max: 10, 
    idleTimeoutMillis: 30000, 
    connectionTimeoutMillis: 5000, 
    });
    ```

<br>
<br>

## RDS Proxy 사용 사례

- 서버리스 애플리케이션 (Lambda, Fargate)
  - AWS Lambda 또는 AWS Fargate와 같은 서버리스 환경에서는 데이터베이스 연결이 빠르게 증가하고 감소합니다.
  - RDS Proxy는 이러한 연결을 효과적으로 관리하여 Cold Start 문제를 줄이고 데이터베이스 부담을 최소화합니다.  

<br>

- 고트래픽 웹 애플리케이션
  - 대규모 웹 애플리케이션에서 다수의 사용자가 동시 접속할 때, RDS Proxy를 활용하면 연결 관리 및 부하 분산을 최적화할 수 있습니다.  

<br>

- 마이크로서비스 아키텍처
  - 여러 마이크로서비스가 동일한 RDS 인스턴스를 사용할 때, RDS Proxy를 통해 효율적인 연결 관리 및 성능 향상이 가능합니다.

<br>

- 읽기/쓰기 분리가 필요한 경우
  - Aurora 클러스터의 Read Replica를 활용하여, 읽기 요청을 자동으로 분배하여 읽기 성능을 향상시킬 수 있습니다.
