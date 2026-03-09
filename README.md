# infra-k8s

Kubernetes 매니페스트 실습 저장소입니다.  
현재는 `k8s_basic` 하위에서 Pod/Service/Deployment/ReplicaSet/Ingress 기초 실습을 정리하고 있습니다.

## 디렉터리 구성

- `k8s_basic/pod_basic/`
  - `nginx_pod.yml`: 단일 Nginx Pod
  - `nginx_pod_busybox.yml`: Nginx + Busybox(`http-pinger`) 멀티 컨테이너 Pod
  - `nginx_service.yml`: `my-nginx` 라벨 대상 Service
- `k8s_basic/multi_pod/`
  - `nginx_deployment.yml`: Deployment(복제본 2개) + Service
  - `nginx_replicaset.yml`: ReplicaSet(복제본 2개) + Service
  - `ingress_nginx.yml`: NGINX Ingress 예제(Host/Path 라우팅)

## 현재 실습 정리

1. Pod 기본
- 단일 Pod 생성/조회/삭제 흐름 확인
- Pod `metadata.labels`와 Service `selector` 매핑 확인

2. Multi-container Pod
- 한 Pod 내 다중 컨테이너 동작 확인
- `http-pinger` 컨테이너에서 `localhost:80` 호출로 Nginx 헬스체크 성격 트래픽 확인

3. Deployment / ReplicaSet
- 선언형 복제본 관리(`replicas`) 확인
- Pod 템플릿(`spec.template`)과 selector 일치의 중요성 확인

4. Service / Ingress
- ClusterIP Service를 통한 내부 접근 확인
- Ingress에서 host/path 기반 라우팅 구성 확인

## 자주 사용하는 명령어

```bash
# 문법/스키마 검증
kubectl apply --dry-run=client -f k8s_basic/

# 서버 검증(클러스터 연결 시)
kubectl apply --dry-run=server -f k8s_basic/

# 변경점 비교
kubectl diff -f k8s_basic/

# 전체 적용
kubectl apply -f k8s_basic/
```

## 참고

- 상세 작업 규칙/검증 기준은 `AGENTS.md`를 따릅니다.
- 실습 후 리소스 정리는 파일 단위로 `kubectl delete -f <file>` 사용을 권장합니다.
