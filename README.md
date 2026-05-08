# basic-1-1

## 1-1 시스템 관제 자동화 환경 구축
본 프로젝트는 서버 장애 시 신속한 원인 분석이 가능하도록 리눅스 기반의 시스템 보안 설정, 권한 체계 설계 및 관제 자동화 스크립트를 구현한 프로젝트입니다. 단순 명령어 실행을 넘어 애플리케이션 배포 환경을 직접 구축하고 시스템 상태를 데이터로 기록하는 엔지니어링 역량을 목표로 합니다.

### 1. 프로젝트 개요
•	학습 시간: 40시간
•	핵심 목표: 다중 사용자 환경 권한 관리, 네트워크 보안 설정, 시스템 리소스 관제 자동화 쉘 스크립트 개발
•	주요 성과: SSH 포트 변경, 방화벽 정책 수립, monitor.sh 개발 및 cron 등록

### 2. 보안 및 네트워크 설정 (Security & Networking)
기본 보안 수칙을 준수하여 서버의 노출을 최소화하고 접근을 통제합니다.
SSH 설정 변경
/etc/ssh/sshd_config 수정
Port 22 -> Port 20022
PermitRootLogin no 설정

sudo service ssh restart
ss -tulnp | grep sshd # 20022 포트 리슨 확인

방화벽 정책 (UFW)
sudo ufw enable
sudo ufw allow 20022/tcp # SSH
sudo ufw allow 15034/tcp # APP
sudo ufw status # 설정 내역 확인

### 3. 계정 및 권한 체계 (Identity & Access Management)
최소 권한의 원칙(Least Privilege)에 따라 협업 환경을 설계했습니다.
•	계정 구성: agent-admin(운영), agent-dev(개발), agent-test(QA)
•	그룹 구성: agent-common(전원), agent-core(admin, dev)
•	디렉토리 정책:
•	upload_files: agent-common 그룹 R/W 가능
•	api_keys & log: agent-core 그룹 전용 R/W 가능

### 4. 시스템 관제 자동화 (Automation)
시스템 상태를 매분 체크하고 로그를 남기는 monitor.sh를 구현했습니다.
핵심 기능
•	프로세스/포트 점검: 앱 서비스 정상 유무 확인 (실패 시 exit 1)
•	리소스 수집: CPU, Memory, Disk 사용률 기록
•	임계값 경고: CPU > 20%, MEM > 10% 등 발생 시 [WARNING] 태그 출력
실행 기록 예시 (monitor.log)
[2026-05-08 15:00:01] PID:48291 CPU:12.5% MEM:5.2% DISK_USED:23%
[2026-05-08 15:01:01] PID:48291 CPU:22.3% MEM:6.0% DISK_USED:23% [WARNING] CPU threshold exceeded

### 5. 실행 및 검증 (How to Run)
본 환경을 검증하기 위한 단계별 명령어입니다.
환경 변수 로드
export AGENT_HOME="/home/agent-admin/agent-app"
export AGENT_PORT=15034
 ... 기타 환경 변수 적용

앱 부트 시퀀스 확인
일반 계정으로 실행
python3 agent_app.py

출력 결과 예시
[1/5] Checking User Account [OK]
...
Agent READY

자동화 등록 (Cron)
agent-admin 계정의 crontab
* * * * * $AGENT_HOME/bin/monitor.sh

## 🛠 Tech Stack
•	OS: Ubuntu 22.04 LTS
•	Script: Bash Shell
•	Language: Python 3 (Application)
•	Tools: UFW, SSH, Crontab, ACL
