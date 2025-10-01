# 📌 GitHub Commit & Branch 전략

본 문서는 프로젝트의 **Branch 전략**과 **Commit 메시지 컨벤션**을 정의합니다.  
모든 팀원이 동일한 규칙을 따름으로써 코드 품질과 협업 효율성을 높이는 것을 목표로 합니다.

---

## 1. Branch 전략

### 1.1 주요 브랜치
```
prod                 : 운영 배포용 안정화 브랜치
develop              : 다음 배포 준비용 통합 브랜치
feature/*            : 기능 개발 브랜치
bugfix/*             : 버그 수정 브랜치 (개발 환경)
hotfix/*             : 긴급 수정 브랜치 (운영 환경)
release/*            : 배포 준비 브랜치
local/*              : 개인 개발/실험용 브랜치
```

### 1.2 브랜치 용도
- **prod**
    - 운영 서버에 배포되는 코드
    - 모든 커밋은 태그(`v1.0.0`)로 관리
- **develop**
    - 다음 배포 후보 코드 통합
    - feature/bugfix 브랜치의 병합 대상
- **feature/***
    - 새로운 기능 개발
    - 네이밍: `feature/{기능명}`
    - 완료 후 develop에 merge
- **bugfix/***
    - QA/개발 환경 버그 수정
    - develop에 merge
- **hotfix/***
    - 운영 긴급 수정
    - prod에서 분기 → 수정 후 prod + develop 동시 merge
- **release/***
    - 배포 직전 안정화
    - QA 완료 후 prod merge + 태그 생성
- **local/***
    - **개인 개발/실험 브랜치**
    - 예: `local/sample`, `local/test-api`
    - **팀 repo에는 push 최소화**
    - 필요 기능은 `feature/*` 브랜치로 분리 후 통합
    - prod/develop에는 직접 merge 금지

---

## 2. 브랜치 플로우

1. **기능 개발**
    - `develop`에서 분기 → `feature/*` → 개발 완료 → PR → `develop` merge

2. **버그 수정**
    - `develop`에서 분기 → `bugfix/*` → 수정 → PR → `develop` merge

3. **운영 긴급 수정**
    - `prod`에서 분기 → `hotfix/*` → 수정 → `prod` + `develop` merge → 태그 생성

4. **배포**
    - `release/*` → QA 완료 → `prod` merge → 태그(`vX.Y.Z`) → 배포

---

## 3. Commit 메시지 컨벤션

### 3.1 기본 형식
```
<type>(<scope>): <subject>
```

- **type**: 작업 유형
- **scope**: 영향 범위(선택)
- **subject**: 간단한 설명 (50자 이내)

### 3.2 Type 정의
| 타입       | 설명 |
|------------|------|
| feat       | 새로운 기능 추가 |
| fix        | 버그 수정 |
| docs       | 문서 수정 (README, Conventions 등) |
| style      | 코드 포맷팅, 세미콜론 누락 등 (비즈니스 로직 영향 없음) |
| refactor   | 코드 리팩토링 (기능 변화 없음) |
| test       | 테스트 코드 추가/수정 |
| chore      | 빌드 설정, 라이브러리 업데이트 등 기타 작업 |
| perf       | 성능 개선 |
| ci         | CI/CD 설정 변경 |

### 3.3 Commit 예시
```bash
feat(user): 회원 가입 API 추가
fix(auth): JWT 토큰 만료 검증 로직 수정
docs(readme): 실행 방법 업데이트
style(controller): import 정렬 및 공백 수정
refactor(order): 주문 결제 로직 서비스 계층으로 이동
test(payment): 결제 성공/실패 단위 테스트 추가
chore(gradle): Spring Boot 3.4.4 버전 업데이트
```

---

## 4. Pull Request (PR) 규칙

1. **브랜치 병합 대상**
    - `feature/*`, `bugfix/*` → `develop`
    - `release/*` → `prod`
    - `hotfix/*` → `prod` + `develop`

2. **코드 리뷰 필수**
    - 최소 1명 이상 승인 후 merge

3. **PR 제목 컨벤션**
   ```
   [feat] 사용자 로그인 API 개발
   [fix] JWT 토큰 만료 버그 수정
   ```

4. **PR 템플릿 예시**
   ```md
   ## 작업 내용
   - 로그인 API 구현
   - JWT 발급/검증 로직 추가

   ## 확인 사항
   - [ ] 단위 테스트 통과
   - [ ] 로컬 빌드 성공
   - [ ] 코드 리뷰 반영 완료
   ```

---

## 5. 태깅 & 릴리즈
- `prod` merge 시 태그 생성
- 태그 네이밍: `v{Major}.{Minor}.{Patch}`
    - `v1.0.0` : 최초 배포
    - `v1.0.1` : 버그 패치
    - `v1.1.0` : 기능 추가
- GitHub Release 자동화 (CHANGELOG 반영)

---

## 6. 프로젝트별 브랜치 기준 시나리오

프로젝트의 성격, 배포 파이프라인, 인프라 운영 방식에 따라 **최상위 기준 브랜치(top branch)** 가 달라질 수 있습니다.  
아래 3가지 패턴 중 프로젝트 상황에 맞는 기준을 선택합니다.

### 6.1 `prod` (prod) 기준 – **전통적인 Git Flow**
- **구조**
  ```
  feature/* → develop → release/* → prod
  hotfix/* → prod (동시에 develop 반영)
  ```
- **특징**
    - `prod` = 운영 서버 배포용 (가장 안정된 코드)
    - `develop` = 다음 배포 준비
    - `release/*` = 배포 전 QA
    - `hotfix/*` = 운영 긴급 수정

### 6.2 `stg` 기준 – **운영 전 안정화 중심**
- **구조**
  ```
  feature/* → dev → stg → prod
  ```
- **특징**
    - `stg` = 최상위 안정화 브랜치 (운영 직전 QA 기준)
    - `prod` = 실제 운영 배포용 (stg에서만 merge)

### 6.3 `dev` 기준 – **간소화 / 속도 중시**
- **구조**
  ```
  feature/* → dev → prod
  ```
- **특징**
    - `dev` = 최상위 브랜치, 모든 기능이 먼저 통합
    - `prod` = 배포 시점에만 merge

---

## 📌 핵심 원칙
- 어떤 브랜치를 최상위로 선택하든 **흐름은 동일**:
  ```
  feature/bugfix → 통합 브랜치(dev/stg) → prod
  ```
- `local/*` 브랜치는 **개인 실험용**으로만 사용, 공식 병합 금지
- 운영(prod) 브랜치는 항상 **태그(vX.Y.Z)**로 버전 관리  