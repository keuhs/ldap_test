# LDAP 연결 테스트 스크립트

LDAP 서버 연결 상태를 테스트하고 검증하는 종합적인 쉘 스크립트입니다. YAML 설정 파일을 통해 유연한 설정 관리가 가능하며, 다양한 테스트 옵션을 제공합니다.

## 주요 기능

- **네트워크 연결 테스트**: LDAP 서버에 네트워크 연결 가능 여부 확인
- **LDAP 인증 테스트**: 제공된 계정으로 LDAP 서버 인증 검증
- **사용자 검색 테스트**: LDAP에서 사용자 검색 기능 테스트
- **그룹 검색 테스트**: LDAP에서 그룹 검색 기능 테스트
- **스키마 정보 조회**: LDAP 서버의 스키마 정보 확인
- **YAML 기반 설정**: 외부 YAML 파일을 통한 설정 관리
- **다양한 실행 모드**: 빠른 테스트, 상세 테스트, 정보 확인 모드
- **색상 코드 로그**: 가독성 높은 로그 출력

## 설치 및 의존성

### 필수 요구사항

- **ldap-utils**: LDAP 클라이언트 도구

```bash
# Ubuntu/Debian
sudo apt-get install ldap-utils

# CentOS/RHEL
sudo yum install openldap-clients

# macOS
brew install openldap
```

### 선택적 요구사항

- **yq**: 고급 YAML 파싱 (권장)

```bash
# Snap
sudo snap install yq

# Direct download
sudo wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64
sudo chmod +x /usr/local/bin/yq

# macOS
brew install yq
```

## 설치 방법

```bash
# 스크립트 다운로드
wget https://raw.githubusercontent.com/your-repo/ldap_test.sh
chmod +x ldap_test.sh

# 또는 Git 클론
git clone https://github.com/your-repo/ldap-test-script.git
cd ldap-test-script
chmod +x ldap_test.sh
```

## 사용 방법

### 기본 사용법

```bash
# 기본 설정 파일 생성
./ldap_test.sh --create-config ldap_config.yaml

# 설정 파일 편집 후 테스트 실행
./ldap_test.sh
```

### 명령행 옵션

```bash
./ldap_test.sh [옵션]

옵션:
  -h, --help                이 도움말 출력
  -c, --config FILE         설정 파일 지정 (기본값: ldap_config.yaml)
  -i, --info                연결 정보만 출력
  -q, --quick               빠른 테스트만 실행
  -v, --verbose             상세한 출력
  --create-config FILE      기본 설정 파일 생성
```

### 사용 예시

```bash
# 기본 설정으로 모든 테스트 실행
./ldap_test.sh

# 지정된 설정 파일로 테스트 실행
./ldap_test.sh -c my_ldap.yaml

# 빠른 연결 테스트만 실행
./ldap_test.sh -q

# 연결 정보만 출력
./ldap_test.sh -i

# 상세한 테스트 실행
./ldap_test.sh -v

# 새로운 설정 파일 생성
./ldap_test.sh --create-config production.yaml
```

## 설정 파일 구조

### 기본 YAML 설정

```yaml
# LDAP 연결 설정
ldap:
  # LDAP 서버 호스트
  host: "ldap.example.com"
  
  # LDAP 서버 포트 (기본값: 389, LDAPS: 636)
  port: 389
  
  # SSL/TLS 사용 여부
  use_ssl: false
  
  # Base DN (검색 시작점)
  base_dn: "dc=example,dc=com"
  
  # Bind DN (인증용 계정)
  bind_dn: "cn=admin,dc=example,dc=com"
  
  # 패스워드
  password: "secure_password"

# 검색 설정 (선택적)
search:
  # 사용자 검색 필터
  user_filter: "(objectClass=person)"
  
  # 그룹 검색 필터
  group_filter: "(objectClass=group)"
  
  # 검색 결과 제한
  size_limit: 100

# 테스트 설정
test:
  # 네트워크 연결 타임아웃 (초)
  timeout: 5
  
  # 상세 출력 여부
  verbose: false
```

### SSL/TLS 설정 예시

```yaml
ldap:
  host: "ldaps.example.com"
  port: 636
  use_ssl: true
  base_dn: "dc=example,dc=com"
  bind_dn: "cn=ldap-user,ou=Service Accounts,dc=example,dc=com"
  password: "secure_password"
```

### Active Directory 설정 예시

```yaml
ldap:
  host: "ad.company.com"
  port: 389
  use_ssl: false
  base_dn: "dc=company,dc=com"
  bind_dn: "cn=ldap-user,ou=Service Accounts,dc=company,dc=com"
  password: "secure_password"

search:
  # Active Directory용 사용자 필터
  user_filter: "(&(objectClass=user)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))"
  
  # Active Directory용 그룹 필터
  group_filter: "(objectClass=group)"
```

## 환경별 설정 관리

### 개발/테스트/운영 환경 분리

```bash
# 각 환경별 설정 파일 생성
./ldap_test.sh --create-config ldap_dev.yaml
./ldap_test.sh --create-config ldap_test.yaml
./ldap_test.sh --create-config ldap_prod.yaml

# 환경별 테스트 실행
./ldap_test.sh -c ldap_dev.yaml
./ldap_test.sh -c ldap_test.yaml
./ldap_test.sh -c ldap_prod.yaml
```

### 배치 테스트 스크립트

```bash
#!/bin/bash
# batch_ldap_test.sh

environments=("dev" "test" "prod")

for env in "${environments[@]}"; do
    echo "========================================="
    echo "Testing $env environment"
    echo "========================================="
    
    if [ -f "ldap_${env}.yaml" ]; then
        ./ldap_test.sh -c "ldap_${env}.yaml" -q
        echo ""
    else
        echo "Configuration file ldap_${env}.yaml not found"
    fi
done
```

## 보안 관리

### 환경 변수 사용

패스워드를 환경 변수로 관리:

```yaml
ldap:
  host: "ldap.example.com"
  port: 389
  use_ssl: false
  base_dn: "dc=example,dc=com"
  bind_dn: "cn=ldap-user,ou=Service Accounts,dc=example,dc=com"
  password: "${LDAP_PASSWORD}"  # 환경 변수 사용
```

실행 시:
```bash
export LDAP_PASSWORD="your_secure_password"
./ldap_test.sh -c ldap_config.yaml
```

### 파일 권한 설정

```bash
# 설정 파일 권한 제한
chmod 600 ldap_config.yaml

# 소유자만 읽기/쓰기 가능
chown $(whoami):$(whoami) ldap_config.yaml
```

## 자동화 및 모니터링

### 크론잡 설정

```bash
# crontab -e
# 매일 오전 9시에 LDAP 연결 테스트 실행
0 9 * * * /path/to/ldap_test.sh -c /path/to/ldap_config.yaml -q >> /var/log/ldap_monitor.log 2>&1

# 매시간 간단한 연결 체크
0 * * * * /path/to/ldap_test.sh -c /path/to/ldap_config.yaml -q > /dev/null 2>&1 || echo "LDAP connection failed at $(date)" >> /var/log/ldap_alerts.log
```

### 로그 파일 관리

```bash
# 결과를 파일로 저장
./ldap_test.sh -c ldap_config.yaml | tee ldap_test_$(date +%Y%m%d_%H%M%S).log

# 로그 로테이션 설정 (/etc/logrotate.d/ldap-test)
/var/log/ldap_monitor.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
}
```

## CI/CD 통합

### Jenkins 파이프라인

```groovy
pipeline {
    agent any
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'test', 'prod'],
            description: 'Select environment to test'
        )
    }
    
    stages {
        stage('LDAP Connection Test') {
            steps {
                script {
                    sh '''
                        chmod +x ldap_test.sh
                        ./ldap_test.sh -c ldap_${ENVIRONMENT}.yaml -q
                    '''
                }
            }
            post {
                failure {
                    emailext (
                        subject: "LDAP Connection Test Failed - ${env.ENVIRONMENT}",
                        body: "LDAP connection test failed for ${env.ENVIRONMENT} environment.",
                        to: "admin@company.com"
                    )
                }
            }
        }
    }
}
```

### GitHub Actions

```yaml
name: LDAP Connection Test

on:
  schedule:
    - cron: '0 9 * * *'  # Daily at 9 AM
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [dev, test, prod]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Install dependencies
      run: |
        sudo apt-get update
        sudo apt-get install -y ldap-utils
    
    - name: Run LDAP test
      run: |
        chmod +x ldap_test.sh
        ./ldap_test.sh -c ldap_${{ matrix.environment }}.yaml -q
      env:
        LDAP_PASSWORD: ${{ secrets.LDAP_PASSWORD }}
```

## Docker 환경

### Dockerfile

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && \
    apt-get install -y ldap-utils wget && \
    rm -rf /var/lib/apt/lists/*

# yq 설치 (선택적)
RUN wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 && \
    chmod +x /usr/local/bin/yq

COPY ldap_test.sh /usr/local/bin/
COPY ldap_config.yaml /etc/ldap/

RUN chmod +x /usr/local/bin/ldap_test.sh

CMD ["/usr/local/bin/ldap_test.sh", "-c", "/etc/ldap/ldap_config.yaml"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  ldap-test:
    build: .
    environment:
      - LDAP_PASSWORD=${LDAP_PASSWORD}
    volumes:
      - ./configs:/etc/ldap/configs:ro
      - ./logs:/var/log/ldap:rw
    command: >
      sh -c "
        /usr/local/bin/ldap_test.sh -c /etc/ldap/configs/ldap_prod.yaml -v
      "
```

## 문제 해결

### 일반적인 오류

1. **연결 거부 오류**
   ```
   ldap_bind: Can't contact LDAP server (-1)
   ```
   - 네트워크 연결 확인
   - 방화벽 설정 확인
   - LDAP 서버 상태 확인

2. **인증 실패**
   ```
   ldap_bind: Invalid credentials (49)
   ```
   - Bind DN과 패스워드 확인
   - 계정 잠금 상태 확인

3. **권한 부족**
   ```
   ldap_search: Insufficient access (50)
   ```
   - 계정의 검색 권한 확인
   - Base DN 설정 확인

### 디버깅 모드

```bash
# 상세한 오류 정보 출력
./ldap_test.sh -v -c ldap_config.yaml

# LDAP 디버그 모드
LDAP_DEBUG=1 ./ldap_test.sh -c ldap_config.yaml
```

## 기여하기

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 라이선스

이 프로젝트는 MIT 라이선스 하에 배포됩니다. 자세한 내용은 `LICENSE` 파일을 참조하세요.

## 지원

- 이슈 리포트: [GitHub Issues](https://github.com/your-repo/ldap-test-script/issues)
- 문의: admin@yourcompany.com

## 변경 로그

### v1.0.0 (2024-12-XX)
- 초기 릴리스
- YAML 설정 파일 지원
- 기본 LDAP 연결 테스트
- 사용자/그룹 검색 기능
- SSL/TLS 지원
- 다양한 실행 모드 제공
