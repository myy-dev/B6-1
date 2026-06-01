# 클라우드 환경에서 웹 서비스 인프라 구축

## 프로젝트 개요

클라우드 환경에서 외부 접속 가능한 웹 서비스 인프라를 직접 구축하는 프로젝트이다.

## 실행환경

- 클라우드 플랫폼: Amazon Web Services
- 리전: 서울 리전 `ap-northeast-2`
- 인스턴스: 프리 티어 대상 micro급 인스턴스 1대
  - 예: `t2.micro` 또는 `t3.micro`
- 운영체제: Ubuntu LTS 또는 Amazon Linux 계열
- 스토리지: EBS 8~10GiB 권장
- 네트워크:
  - VPC 1개
  - Public Subnet 1개
  - Internet Gateway 1개
  - Route Table에 `0.0.0.0/0 -> Internet Gateway` 경로 설정
- 웹 서버: Nginx 등 HTTP 요청에 200 응답을 반환할 수 있는 웹 서버
- 접근 방식:
  - SSH: 개인 IP 또는 지정 IP 대역에서만 허용
  - HTTP: `0.0.0.0/0`에서 80번 포트 접근 허용

## 프로젝트 구조

```text
B6-1/
├── README.md
└── docs/
    ├── architecture.(png|pdf)
    ├── troubleshooting.(md|pdf)
    ├── cleanup-checklist.md
    └── screenshots/
        ├── external-access.png
        ├── docker-ps.png
        └── billing-or-resources.png
```

> `docs/` 디렉터리는 과제 산출물 제출을 위해 생성한다. 스크린샷 파일명은 실제 제출 자료에 맞게 변경할 수 있다.

## 수행 항목 체크리스트

### 네트워크 구성

- [ ] VPC 1개 생성
- [ ] Public Subnet 1개 생성
- [ ] Internet Gateway VPC 연결
- [ ] Public Subnet Route Table에 `0.0.0.0/0 -> Internet Gateway` 경로 추가
- [ ] Public Subnet 인스턴스의 인터넷 아웃바운드 통신 확인
  - [ ] 예: `curl https://example.com` 성공

### 컴퓨트 및 웹 서버 배포

- [ ] Public Subnet 내 EC2 인스턴스 1대 생성
- [ ] SSH 인스턴스 접속
- [ ] Nginx 등 웹 서버 설치
- [ ] 웹 서버 실행 상태 확인
- [ ] 인스턴스 내부 `curl http://localhost` 200 응답 확인

### 접근 제어

- [ ] Security Group 인바운드 필요 포트만 허용
- [ ] HTTP 80번 포트 `0.0.0.0/0` 접근 허용
- [ ] SSH 22번 포트 개인 IP 또는 지정 IP 대역 접근 허용
- [ ] `0.0.0.0/0` 대상 전체 포트 `0-65535` 허용 규칙 미생성

### IAM 최소 권한

- [ ] 실습용 IAM 사용자 또는 Role 1개 사용
- [ ] EC2, VPC, Security Group 구성 필요 권한 제한
- [ ] 생성, 조회, 태그, 연결 등 실습 필요 권한 구성
- [ ] 실습 무관 S3, RDS 등 서비스 권한 미부여
- [ ] 관리자 권한(`AdministratorAccess`) 권한 미부여

### 외부 접속 검증

- [ ] A 또는 B 중 외부 접속 검증 방식 선택
- [ ] A 선택 시 브라우저 `http://<퍼블릭 IP>` 페이지 정상 표시
- [ ] B 선택 시 `GET http://<퍼블릭 IP>/health` 200 응답
- [ ] 고정 응답 문구 확인
  - [ ] 예: `OK`, `Hello Cloud`, `Welcome to nginx!`
- [ ] 선택한 검증 방식 README 명시

### 운영 안정성 및 과금 방지

- [ ] 실습 종료 후 생성 리소스 정리
- [ ] EC2 종료 또는 삭제 상태 확인
- [ ] EBS 볼륨 삭제 여부 확인
- [ ] Elastic IP 사용 시 Release 여부 확인
- [ ] Internet Gateway Detach 및 삭제 여부 확인
- [ ] VPC, Subnet, Route Table 삭제 여부 확인
- [ ] 정리 결과 `docs/cleanup-checklist.md` 기록

### 보너스 과제

#### HTTPS 적용

- [ ] 무료 서브도메인, 보유 도메인 또는 구매 도메인 연결
- [ ] Let’s Encrypt 등 HTTPS 인증서 적용
- [ ] HTTPS 접속 정상 동작 확인
- [ ] 인증서 적용 방식과 검증 결과 문서화

#### Docker 컨테이너 웹 서비스 배포

- [ ] EC2 인스턴스 내부 Docker 설치
- [ ] 컨테이너 웹 서비스 실행
- [ ] 사용할 컨테이너 이미지 선택
  - [ ] 이전 미션 이미지 또는 공개 이미지 사용 가능
- [ ] 인스턴스 내부 `curl http://localhost` 정상 응답 확인
- [ ] 외부 `http://<퍼블릭 IP>` 접속 또는 `/health` 호출 정상 응답 확인
- [ ] README 컨테이너 이미지명과 실행 방식 기록
- [ ] README 포트 매핑 정보 기록
- [ ] `docker ps` 컨테이너 Up 상태 확인 및 스크린샷 저장
- [ ] 외부 접속 결과 스크린샷 저장
- [ ] Docker 검증 결과 스크린샷 2장 이상 제출

## 제약 사항

- 모든 리소스 AWS 서울 리전 `ap-northeast-2` 생성
- 프리 티어 범위 내 실습
- 루트 계정 사용 금지
- 별도 IAM 사용자 또는 Role 콘솔 접근
- EC2, VPC, Security Group 실습 필요 권한 제한
- `AdministratorAccess` 권한 사용 금지
- 키페어 1개 생성 및 안전 보관
- 키페어 재발급 불가 전제 관리
- SSH 22번 포트 본인 개인 IP 또는 지정 IP 대역만 허용
- 운영 또는 관리 목적 포트 방치 금지
- `0.0.0.0/0` 대상 전체 포트 허용 규칙 생성 금지
- 생성 리소스 정리 대상 추적
- 실습 종료 후 모든 리소스 종료 또는 삭제
- 정리 대상: EC2, EBS Volume, Elastic IP, Internet Gateway, VPC
- 생성 시 NAT Gateway, ELB/ALB, RDS 삭제 확인
- 모든 리소스 삭제 후 Billing Dashboard 확인 권장

## 외부 접속 검증 기록

- 검증 방식: A 또는 B 중 선택
- 접속 URL 또는 IP: `http://<퍼블릭 IP>`
- 응답 결과:
- 스크린샷 경로:

## Docker 보너스 기록

- 컨테이너 이미지:
- 실행 방식:
- 포트 매핑:
- 내부 검증 결과:
- 외부 검증 결과:
- 스크린샷 경로:
