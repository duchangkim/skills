# k3s·ArgoCD 운영

운영 명령은 `ssh homelab kubectl --context default ...`로 실행한다.

## GitOps 구조에서 주의할 점

- `argocd/applications/`가 App-of-Apps(`root-app`) 대상이다. 여기 Application을 추가·삭제하면 앱이 생성·삭제된다(`prune: true`).
- Application에는 `resources-finalizer`가 있어 삭제 시 PVC 등 하위 리소스도 함께 지워진다. 삭제 전 사용자 확인을 받는다.
- `infrastructure` 앱은 `prune: false`다. 저장소에서 지운 인프라 리소스는 클러스터에서 직접 삭제해 마무리한다.
- 이미지는 argocd-image-updater가 Application 어노테이션 기준으로 갱신한다. 이미지를 바꾸기 전 대상 여부를 확인한다.

## 앱 추가

기존 앱의 매니페스트와 Application을 따라 작성한다. 리소스 제한, Sealed Secret([sealed-secrets.md](sealed-secrets.md)), 외부 공개([cloudflare.md](cloudflare.md))를 함께 처리한다. 가능하면 로컬 k3d에서 먼저 검증한다.

## 장애 진단

Application 동기화 오류 → 파드 상태·이벤트·이전 로그 → 노드 자원(`OOMKilled`, `Pending`) → Service·Ingress·인증서·cloudflared 순으로 범위를 좁힌다.

같은 revision으로 자동 동기화 재시도 한도에 걸리면 원인 수정 커밋을 push하거나 수동 Sync한다. CRD annotation 크기 초과 오류는 저장소의 `docs/argocd-crd-sync-troubleshooting.md`를 따른다.
