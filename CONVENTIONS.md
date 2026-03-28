# Org Conventions

alphachu-volleyball 전체 repo에 적용되는 공통 컨벤션.

각 repo의 `CLAUDE.md`는 이 문서를 기반으로 하되, repo 고유 사항을 추가/오버라이드한다.

---

## 버전 관리

- **Semantic Versioning (semver)** 준수: `MAJOR.MINOR.PATCH`
- Git tag로 버전 표기 (예: `v0.1.0`)
- 각 repo는 독립적으로 버전을 관리한다
- repo 간 의존성은 Git tag pinning으로 버전을 명시한다

## 브랜치 전략

| 브랜치 | 용도 | 머지 방식 |
|--------|------|-----------|
| `main` | 안정 릴리스 상태 유지 | `release/*` → main: merge commit |
| `release/{version}` | 릴리스 단위 통합 | `feat/*`, `fix/*` → release: squash merge |
| `feat/*`, `fix/*` | 기능/버그 단위 작업 | PR로 release 브랜치에 머지 |

워크플로우: `feat/*` → `release/{version}` (squash) → `main` (merge commit) → tag

`.github` repo 등 문서 전용 repo는 release branch 없이 `feat/*` → `main` (squash merge)로 직접 머지한다.

## 커밋 컨벤션

[Conventional Commits](https://www.conventionalcommits.org/) 형식을 따른다.

```
<type>(<scope>): <subject>

feat(env): add random ball position mode
fix(wrapper): correct observation space shape
docs(readme): update architecture diagram
chore(ci): add ruff lint workflow
```

주요 type: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`

## Python repo 공통

### 환경 관리

- **Python**: 3.10+
- **패키지 관리**: uv (`pyproject.toml` + `uv.lock`)
- `uv lock`으로 lock 파일 관리, `uv sync`로 설치

### 코드 품질

- **Linter/Formatter**: ruff
- **테스트**: pytest
- ruff, pytest는 CI에서 자동 실행

### ruff 기본 설정

각 repo의 `pyproject.toml`에 포함:

```toml
[tool.ruff]
target-version = "py310"
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "I", "UP"]
```

## JavaScript/Web repo 공통

- Linter: eslint
- 패키지 관리: npm 또는 yarn (repo별 결정)

## CI/CD

### Python repo (GitHub Actions)

| 트리거 | 내용 |
|--------|------|
| PR, push to main | ruff lint, pytest |
| tag push (`v*`) | (해당 시) release 생성 |

- 학습은 CI에서 실행하지 않는다 (GPU 필요, 장시간 소요)

### Web repo (GitHub Actions)

| 트리거 | 내용 |
|--------|------|
| PR, push to main | eslint, 빌드 확인 |
| push to main | GitHub Pages 배포 (world-tournament) |

## Repo 간 의존성

### pika-zoo → training-center

training-center는 pika-zoo를 Git dependency로 참조하며, tag를 pinning한다:

```toml
[project]
dependencies = [
  "pika-zoo @ git+https://github.com/alphachu-volleyball/pika-zoo@v0.1.0",
]
```

pika-zoo 버전 업데이트 시 training-center의 dependency도 함께 업데이트한다.

### training-center → world-tournament

- training-center에서 ONNX 모델을 GitHub Releases artifact로 배포
- world-tournament에서 해당 release asset을 fetch하여 사용
- PyPI, HuggingFace Hub 등 외부 레지스트리는 현재 사용하지 않음

## Artifact 관리

### 모델 파일

- Git에 직접 커밋하지 않는다 (수십~수백 MB)
- training-center의 GitHub Releases에 ONNX 모델 첨부
- `models/checkpoints/`, `models/exported/`는 `.gitignore`에 포함

### 게임 에셋 (스프라이트, 사운드)

- 파일이 작으므로 Git에 직접 커밋

### 실험 추적

- W&B 또는 TensorBoard 연동 (SB3 네이티브 지원)
- 하이퍼파라미터, 보상 커브, ELO 추적에 활용

## 코드 복사 방침

- 서브모듈 사용하지 않음 — 상당한 커스터마이징 필요
- 외부 코드를 복사할 때는 반드시 다음을 포함:
  - 원본 출처 URL
  - 라이선스 파일 (LICENSE)
  - 변경사항 기록 (ATTRIBUTION.md)

### 참고 소스

| 소스 | 복사 위치 | 라이선스 |
|------|-----------|----------|
| [helpingstar/pika-zoo](https://github.com/helpingstar/pika-zoo) | pika-zoo repo | MIT |
| [hankluo6/gym-pikachu-volleyball](https://github.com/hankluo6/gym-pikachu-volleyball) | pika-zoo repo (참고) | 확인 필요 |
| [gorisanson/pikachu-volleyball](https://github.com/gorisanson/pikachu-volleyball) | world-tournament repo (fork) | UNLICENSED (확인 필요) |

## 하드웨어 참고

- AMD Ryzen 7 3700X (8C/16T), NVIDIA RTX 2080 Super (8GB)
- 저차원 벡터 관측 + MLP 정책이라 GPU보다 CPU(환경 병렬화)가 병목
- SB3 `SubprocVecEnv`로 8~16개 환경 병렬 실행 가능

## 라이선스

- pika-zoo: MIT License
- pikachu-volleyball (원본): UNLICENSED (확인 필요)
- 원작 게임 에셋: "(C) SACHI SOFT / SAWAYAKAN Programmers", "(C) Satoshi Takenouchi" 1997