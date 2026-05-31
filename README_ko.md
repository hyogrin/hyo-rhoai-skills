# hyo-rhoai-skills

Red Hat OpenShift AI (RHOAI) 모델 배포 및 운영을 위한 실전 AI 스킬 모음입니다.

OpenShift AI에서 모델 서빙 시 실제로 겪는 문제들 — 특히 KServe storage-initializer의 HuggingFace 모델 다운로드가 중간에 실패하는 문제 — 을 해결합니다.

## 설치 방법

### Lola 사용 (추천)

```bash
# 모듈 추가
lola mod add https://github.com/hyogrin/hyo-rhoai-skills.git

# 현재 프로젝트의 Cursor에 설치
lola install hyo-rhoai-skills -a cursor

# 전역 설치 (모든 프로젝트에서 사용)
lola install hyo-rhoai-skills -a cursor --scope user
```

### .lola-req 사용 (선언형 관리)

프로젝트의 `.lola-req` 파일에 추가:

```
https://github.com/hyogrin/hyo-rhoai-skills.git@main
```

그 다음:

```bash
lola sync
```

### 수동 설치

```bash
# 스킬 파일을 프로젝트에 복사
cp -r skills/hf-model-deploy /path/to/your-project/.cursor/skills/
```

## 제공 스킬

| 스킬 | 설명 |
|------|------|
| **[hf-model-deploy](skills/hf-model-deploy/SKILL.md)** | 안정적이고 실패에 강한 RHOAI 모델 배포 |

### /hf-model-deploy

RHOAI 모델 서빙의 가장 큰 고충: **불안정한 모델 가중치 다운로드**를 해결합니다.

**전략 우선순위:**

1. **OCI ModelCar** — 모델을 컨테이너 이미지로 (시작 시 다운로드 제로)
2. **S3/MinIO Pre-Cache** — 한 번 다운, 여러 번 서빙 (엔터프라이즈급)
3. **PVC Pre-Cache** — 로컬 볼륨 캐싱 (단일 Pod용)
4. **Direct hf://** — 10GB 초과 모델에 사용 금지 (hang, resume 불가)

**포함된 내용:**
- OCI ModelCar 카탈로그 검색 (quay.io API)
- S3/MinIO 전체 구성 + HF→S3 전송 Job
- PVC 기반 다운로드 (`hf_transfer` 고속 병렬, 이어받기 지원)
- GPU VRAM vs 모델 크기 가이드
- 주요 모델 패밀리별 Tool Calling 설정법
- 일반적인 배포 실패 트러블슈팅 가이드

## 보완 관계

이 모듈은 Red Hat Ecosystem Engineering의 [rh-ai-engineer](https://github.com/RHEcosystemAppEng/agentic-collections)와 함께 사용하도록 설계되었습니다:

```bash
# 두 모듈을 함께 설치하면 완벽한 커버리지
lola install -f rh-ai-engineer -a cursor  # 배포 워크플로
lola install hyo-rhoai-skills -a cursor   # 모델 획득
```

| 관심사 | rh-ai-engineer | hyo-rhoai-skills |
|--------|---------------|-----------------|
| 모델 획득 (가중치 다운로드) | ❌ 모델이 이미 있다고 가정 | ✅ 전체 다운로드 전략 |
| 배포 워크플로 (InferenceService) | ✅ 완전한 워크플로 | 기본 매니페스트 |
| 런타임 선택 (vLLM/NIM/Caikit) | ✅ 대화형 | vLLM 중심 |
| 다운로드 실패 트러블슈팅 | ❌ | ✅ XET, timeout, OOM 해결 |
| GPU 사이징 가이드 | 일부 (references) | ✅ 완전한 테이블 |

## 전제 조건

- RHOAI가 설치된 OpenShift 클러스터
- `oc` CLI 인증 완료
- GPU 노드 (NVIDIA)
- [Lola](https://github.com/RedHatProductSecurity/lola) (설치용, 선택)

## 검증된 환경

| 플랫폼 | 버전 | GPU |
|--------|------|-----|
| OpenShift | 4.16+ | L4, L40S, A100 |
| RHOAI | 2.16+ / 3.0+ | NVIDIA |
| vLLM | 0.6.x+ (Red Hat 이미지) | CUDA 12+ |

## 프로젝트 구조

```
hyo-rhoai-skills/
├── README.md                              # 영문 README
├── README_ko.md                           # 한국어 README (이 파일)
├── skills/
│   └── hf-model-deploy/
│       ├── SKILL.md                       # 메인 스킬 정의
│       └── ansible-role-example.yml       # Ansible Role 참고 구현
└── .gitignore
```

## 기여 방법

1. 이 레포를 Fork
2. `skills/<skill-name>/SKILL.md` 아래에 스킬 생성
3. [Lola 스킬 포맷](https://lobstertrap.org/lola/concepts/skills-and-modules/) 따르기
4. `lola mod add ./ && lola install hyo-rhoai-skills -a cursor`로 테스트
5. PR 제출

## 다른 프로젝트에서 사용하기

```bash
# 방법 1: lola로 전역 설치 (모든 프로젝트에 적용)
lola install hyo-rhoai-skills -a cursor --scope user

# 방법 2: 특정 프로젝트에만 설치
cd /path/to/my-project
lola install hyo-rhoai-skills -a cursor

# 방법 3: .lola-req로 프로젝트 의존성 관리
echo "https://github.com/hyogrin/hyo-rhoai-skills.git@main" >> .lola-req
lola sync

# 방법 4: 업데이트
lola mod update hyo-rhoai-skills && lola update
```

## 라이선스

Apache License 2.0
