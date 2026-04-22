# infra-practice

인프라 학습용 연습 프로젝트. FastAPI로 Hello World 앱을 만들고 Docker → CI/CD → AWS EC2 배포까지 전 과정을 직접 돌려보는 게 목표.

## 전제 조건

- 실제 사용자 없음 (순수 학습 목적)
- 비용: **프리티어 또는 최소비용만** 사용
- 스택: Python 3.12 + FastAPI

## 전체 로드맵

### 1단계. 로컬 Docker화 (← 지금 여기)

앱을 Docker 이미지로 빌드하고 로컬에서 컨테이너로 실행해보기.

- FastAPI 앱 작성 (`main.py`)
- `Dockerfile` 작성
- 로컬에서 빌드 & 실행

```bash
# 이미지 빌드
docker build -t infra-practice:latest .

# 컨테이너 실행
docker run -p 8000:8000 infra-practice:latest

# 확인
curl http://localhost:8000/
curl http://localhost:8000/health
```

### 2단계. 이미지 레지스트리 (Docker Hub)

빌드한 이미지를 Docker Hub에 푸시해서 어디서든 받아쓸 수 있게.

- Docker Hub 계정 만들기 (무료)
- `docker tag` → `docker push`
- ECR은 500MB까지만 무료라 학습 단계에선 Docker Hub 권장

### 3단계. AWS EC2 인스턴스 준비

- AWS 계정 생성 (결제수단 등록 필요하지만 프리티어 내에선 과금 없음)
- EC2 **t2.micro 또는 t3.micro** 1대 (12개월 무료, 750시간/월)
- Amazon Linux 2023 또는 Ubuntu
- 보안그룹: 22(SSH), 8000(앱) 오픈
- EBS 30GB 이하 (초과분 과금)
- Docker 설치

### 4단계. 수동 배포 (개념 잡기)

SSH로 접속 → `docker pull` → `docker run`. 한 번 수동으로 성공시키는 게 CI/CD 자동화의 출발점.

### 5단계. GitHub Actions CI/CD

main 브랜치에 push하면 자동으로:

1. Docker 이미지 빌드
2. Docker Hub에 푸시
3. EC2에 SSH로 접속해서 최신 이미지 pull & 재시작

### 6단계 (선택). 점진적 개선

- Nginx 리버스 프록시
- HTTPS (Let's Encrypt / Certbot)
- 도메인 연결 (Route53 호스팅존은 월 $0.50, 이 시점부터 유료)
- 무중단 배포
- DB 붙이기 (EC2 안에 docker-compose로 PostgreSQL)
- 나중에 ECS / Kubernetes

## 비용 관련 주의사항

프리티어 범위 내에서 쓰기 위한 체크리스트:

| 항목 | 가이드 |
|---|---|
| EC2 인스턴스 타입 | **t2.micro 또는 t3.micro 만** |
| EBS 볼륨 | 30GB 이하 |
| Elastic IP | 인스턴스에 **붙어있을 때만 무료**, 떼놓으면 시간당 과금 |
| NAT Gateway | **사용 금지** (프리티어 아님) |
| ELB (로드밸런서) | **사용 금지** (학습 단계엔 EC2 퍼블릭 IP 직접 사용) |
| Route53 호스팅존 | 월 $0.50 — 도메인 필요해질 때까지 스킵 |
| RDS | 쓸 거면 db.t3.micro 프리티어. 아니면 EC2 안에 Docker로 DB 같이 띄우기 |

## DB 관련

현재는 DB 없음. 필요해지면:

- **가장 저렴/간단**: EC2 안에 docker-compose로 PostgreSQL 같이 올리기 (추가 비용 0)
- **프리티어**: RDS db.t3.micro (12개월 무료, 이후 월 $15~)
- **초간단**: SQLite 파일

## 프로젝트 구조

```
infra-practice/
├── main.py              # FastAPI 앱
├── requirements.txt     # Python 의존성
├── Dockerfile           # 이미지 빌드 정의
├── .dockerignore        # 이미지에 넣지 않을 파일
├── .gitignore
└── README.md
```

## 엔드포인트

| Method | Path | Response |
|---|---|---|
| GET | `/` | `{"message": "Hello World!"}` |
| GET | `/health` | `{"status": "ok"}` |
