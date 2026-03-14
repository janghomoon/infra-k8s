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

## 8) 현재 매니페스트 리소스 현황 (2026-03-12 기준)
현재 저장소의 Kubernetes 매니페스트(`k8s_basic/`, `ordersystrem/k8s/`, `msa/k8s/`, `msa/*/k8s/`) 기준 리소스 목록:

### `k8s_basic/`
- `k8s_basic/pod_basic/nginx_pod.yml`
  - `Pod/my-nginx` (`namespace: jhm-dev`)
- `k8s_basic/pod_basic/nginx_pod_busybox.yml`
  - `Pod/my-nginx2` (`namespace: jhm-dev`, 2개 컨테이너: `nginx`, `http-pinger`)
- `k8s_basic/pod_basic/nginx_service.yml`
  - `Service/my-service` (`namespace: jhm-dev`)
- `k8s_basic/multi_pod/nginx_deployment.yml`
  - `Deployment/nginx-deployment` (`namespace: jhm-dev`)
  - `Service/nginx-service` (`namespace: jhm-dev`)
- `k8s_basic/multi_pod/nginx_replicaset.yml`
  - `ReplicaSet/nginx-replicaset` (`namespace: jhm-dev`)
  - `Service/nginx-service` (`namespace: jhm-dev`)
- `k8s_basic/multi_pod/ingress_nginx.yml`
  - `Ingress/nginx-ingress` (`namespace: jhm-dev`, host: `server.jhm-dev.click`)

### `ordersystrem/k8s/k8s-ordersystem/`
- `depl_svc.yml`
  - `Deployment/ordersystem-backend` (`namespace: jhm-dev`)
  - `Service/ordersystem-backend-service` (`namespace: jhm-dev`)
- `hpa.yml`
  - `HorizontalPodAutoscaler/ordersystem-backend-hpa` (`namespace: bradkim`)
- `ingress.yml`
  - `Ingress/order-backend-ingress` (`namespace: jhm-dev`, host: `server.jhm-dev.click`)
- `https.yml`
  - `ClusterIssuer/my-issuer`
  - `Certificate/server-jhm-click-tls` (`namespace: jhm-dev`)
- `redis.yml`
  - `Deployment/redis` (`namespace: jhm-dev`)
  - `Service/redis-service` (`namespace: jhm-dev`)

### `ordersystrem/k8s/k8s-argocd/`
- `argocd-application.yaml`
  - `Application/order-backend` (`namespace: argocd`)
- `argocd-service.yml`
  - `Service/argocd-server` (`namespace: argocd`)
- `argocd-ingress.yml`
  - `Ingress/argocd-server-ingress` (`namespace: argocd`, host: `argo.bradkim197.shop`)
- `https.yml`
  - `Certificate/argo-bradkim197-com-tls` (`namespace: argocd`)

### `ordersystrem/k8s/k8s-monitoring/`
- `prometheus-config.yml`
  - `ConfigMap/prometheus-config` (`namespace: monitoring`)
- `prometheus-rbac.yml`
  - `ClusterRole/prometheus`
  - `ServiceAccount/prometheus` (`namespace: monitoring`)
  - `ClusterRoleBinding/prometheus`
- `prometheus-depl_svc.yml`
  - `Deployment/prometheus` (`namespace: monitoring`)
  - `Service/prometheus-service` (`namespace: monitoring`)
- `grafana-depl_svc.yml`
  - `Deployment/grafana` (`namespace: monitoring`)
  - `Service/grafana` (`namespace: monitoring`)
- `node_exporter.yml`
  - `DaemonSet/node-exporter` (`namespace: monitoring`)

### `msa/k8s/`
- `redis.yml`
  - `Deployment/redis` (`namespace: bradkim`)
  - `Service/redis-service` (`namespace: bradkim`)
- `zookeeper.yml`
  - `Deployment/zookeeper` (`namespace: bradkim`)
  - `Service/zookeeper-service` (`namespace: bradkim`)
- `kafka.yml`
  - `Deployment/kafka` (`namespace: bradkim`)
  - `Service/kafka-service` (`namespace: bradkim`)
- `ingress.yml`
  - `Ingress/order-backend-ingress` (`namespace: bradkim`, host: `server.bradkim197.shop`)
- `ingress_without_gateway.yml`
  - `Ingress/order-backend-ingress` (`namespace: bradkim`, host: `server.bradkim198.shop`)
- `https.yml`
  - `ClusterIssuer/my-issuer`
  - `Certificate/server-bradkim197-com-tls` (`namespace: bradkim`)

### `msa/*/k8s/`
- `msa/apigateway/k8s/depl_svc.yml`
  - `Deployment/apigateway-depl` (`namespace: bradkim`)
  - `Service/apigateway-service` (`namespace: bradkim`)
- `msa/member/k8s/depl_svc.yml`
  - `Deployment/member-depl` (`namespace: bradkim`)
  - `Service/member-service` (`namespace: bradkim`)
- `msa/product/k8s/depl_svc.yml`
  - `Deployment/product-depl` (`namespace: bradkim`)
  - `Service/product-service` (`namespace: bradkim`)
- `msa/ordering/k8s/depl_svc.yml`
  - `Deployment/ordering-depl` (`namespace: bradkim`)
  - `Service/ordering-service` (`namespace: bradkim`)

참고:
- `ClusterIssuer`, `ClusterRole`, `ClusterRoleBinding`은 cluster-scoped 리소스입니다.
- 일부 매니페스트에서 `ClusterIssuer`에 `metadata.namespace`가 포함되어 있으나, cluster-scoped 리소스 특성상 적용 시 무시됩니다.
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

## 10) 최근 이슈/수정 요약 (2026-03-15)
### Argo CD ApplicationSet Controller CrashLoopBackOff
- 증상
  - `argocd/argocd-applicationset-controller-*` Pod가 `0/1 CrashLoopBackOff` 반복
- 직접 원인
  - `applicationsets.argoproj.io` CRD 누락
  - 로그 핵심: `no matches for kind "ApplicationSet" in version "argoproj.io/v1alpha1"`
- 확인 명령
```bash
kubectl get crd applicationsets.argoproj.io
kubectl logs -n argocd <applicationset-controller-pod> --previous
kubectl get events -n argocd --sort-by=.metadata.creationTimestamp | tail -n 30
```
- 복구 명령(버전 고정 권장)
```bash
kubectl apply -k "https://github.com/argoproj/argo-cd/manifests/crds?ref=v3.3.3"
kubectl rollout restart deployment/argocd-applicationset-controller -n argocd
kubectl rollout status deployment/argocd-applicationset-controller -n argocd
kubectl get pods -n argocd
```
- 참고
  - `https://raw.githubusercontent.com/argoproj/argo-cd/v3.3.3/manifests/crds/install.yaml` 경로는 404 발생 가능
  - 이 경우 `-k .../manifests/crds?ref=v3.3.3` 방식 사용

### 스케줄링 경고 분리 해석
- `FailedScheduling: Too many pods` / `untolerated taint`는 초기 스케줄 지연 원인
- 컨테이너 기동 후 반복 종료(BackOff)와는 별개로 분리 분석 필요

### Argo CD Application 경로 주의사항
- 파일: `ordersystrem/k8s/k8s-argocd/argocd-application.yaml`
- `apiVersion: argoproj.io/v1alpha1`, `kind: Application` 자체는 정상
- `spec.source.path`는 실제 저장소 경로(`ordersystrem/...`)와 일치해야 함
