# AGENTS.md

infra-k8s 저장소 작업 가이드입니다. 이 파일의 목적은 에이전트/기여자가 같은 방식으로 Kubernetes 매니페스트를 작성, 검증, 커밋하도록 기준을 맞추는 것입니다.

## 1) 기본 원칙
- 변경은 작고 명확하게 나눈다. (리소스 1~3개 단위 권장)
- 실제 적용 전 `dry-run` 또는 `diff`로 검증한다.
- 환경별 파일을 분리하고, 공통 리소스와 혼합하지 않는다.
- 민감 정보(토큰, 키, 비밀번호)는 평문 커밋하지 않는다.

## 2) 디렉터리 규칙
- 현재 기본 학습/예제 경로: `k8s_basic/`
- 권장 구조:
  - `k8s_basic/pod_basic/`: Pod 기초 예제
  - `apps/<app-name>/base/`: 공통 매니페스트
  - `apps/<app-name>/overlays/<env>/`: 환경별 오버레이(dev/stg/prod)
- 새 파일은 목적이 드러나는 이름으로 생성한다.
  - 예: `nginx-deployment.yaml`, `api-service.yaml`

## 3) 매니페스트 작성 규칙
- `metadata.name`은 소문자/하이픈(kebab-case) 사용
- `labels`는 최소한 아래 키 유지 권장
  - `app.kubernetes.io/name`
  - `app.kubernetes.io/part-of`
  - `app.kubernetes.io/component`
- 이미지 태그는 `latest` 대신 고정 버전 사용 권장
- `resources.requests/limits`를 가능한 명시
- 프로브(`livenessProbe`, `readinessProbe`) 설정 우선 고려

## 4) 검증 명령
로컬 클러스터 컨텍스트 기준:

```bash
# 문법/스키마 기본 검증
kubectl apply --dry-run=client -f <file-or-dir>

# 서버 검증 가능 시
kubectl apply --dry-run=server -f <file-or-dir>

# 실제 반영 전 변경점 확인
kubectl diff -f <file-or-dir>
```

디렉터리 단위 적용 예:

```bash
kubectl apply -f k8s_basic/
```

## 5) 커밋 규칙
- 커밋 메시지는 목적 중심으로 작성
  - 예: `feat(k8s): add nginx pod example`
  - 예: `fix(k8s): correct service selector label`
- 하나의 커밋에는 하나의 논리적 변경만 포함
- `.idea/` 같은 IDE 산출물은 버전 관리 제외 권장

## 6) PR/리뷰 체크리스트
- 리소스 이름/라벨 규칙을 지켰는가
- 이미지 태그가 고정되어 있는가
- `dry-run`/`diff` 결과를 확인했는가
- 운영 영향(재시작, 다운타임 가능성)을 설명했는가
- 롤백 방법(`kubectl rollout undo` 등)을 명시했는가

## 7) 금지/주의 사항
- 운영 환경에 `kubectl apply -f .` 같은 광역 명령 지양
- 테스트되지 않은 변경을 한 번에 대량 반영 금지
- Secret 평문 커밋 금지 (필요 시 External Secrets, sealed-secrets 등 사용)

## 8) 현재 예제 리소스 현황 (2026-03-10 기준)
현재 `k8s_basic/` 디렉터리에 있는 매니페스트 기준 리소스 목록:

- `k8s_basic/pod_basic/nginx_pod.yml`
  - `Pod/my-nginx` (`namespace: jhm-dev`)
- `k8s_basic/pod_basic/nginx_pod_busybox.yml`
  - `Pod/my-nginx2` (`namespace: jhm-dev`, 2개 컨테이너: `nginx`, `http-pinger`)
- `k8s_basic/pod_basic/nginx_service.yml`
  - `Service/my-service` (`namespace: jhm-dev`, selector: `app=my-nginx`)

- `k8s_basic/multi_pod/nginx_deployment.yml`
  - `Deployment/nginx-deployment` (`namespace: jhm-dev`)
  - `Service/nginx-service` (`namespace: jhm-dev`)
- `k8s_basic/multi_pod/nginx_replicaset.yml`
  - `ReplicaSet/nginx-replicaset` (`namespace: jhm-dev`)
  - `Service/nginx-service` (`namespace: jhm-dev`)
- `k8s_basic/multi_pod/ingress_nginx.yml`
  - `Ingress/nginx-ingress` (`namespace: jhm-dev`, host: `server.jhm-dev.click`)

참고:
- 일부 예제 파일명(`*_pod.yml`)은 기존 학습 파일명을 유지 중입니다.
- 운영 적용 전에는 4) 검증 명령의 `dry-run`/`diff`를 우선 실행합니다.

## 9) 실습 진행 상태 요약
- `pod_basic`
  - 단일 Pod 생성 및 Service 연결 실습 완료
  - Multi-container Pod(sidecar 형태: `nginx` + `http-pinger`) 실습 완료
- `multi_pod`
  - Deployment 기반 다중 Pod 관리 및 Service 라우팅 실습 완료
  - ReplicaSet 기반 다중 Pod 관리 실습 파일 보유
  - Ingress(host/path 기반 라우팅) 실습 파일 보유
