# Cloudflare·외부 공개

외부 트래픽은 `Cloudflare(DNS·Access) → Tunnel → cloudflared(kube-system) → Traefik(websecure) → Service` 경로만 사용한다.

- Tunnel은 원격 관리형이다. 퍼블릭 호스트네임 매핑은 Zero Trust 대시보드가 정본이고, 클러스터에는 토큰만 Sealed Secret으로 있다.
- 인증서는 cert-manager `ClusterIssuer` `letsencrypt-prod`(HTTP-01, Traefik)로 발급한다.

## 서브도메인 공개

1. 기존 앱 Ingress를 따라 Ingress를 추가한다(`ingressClassName: traefik`, `letsencrypt-prod` 어노테이션, `websecure` 엔트리포인트, TLS).
2. 대시보드에서 Tunnel에 퍼블릭 호스트네임을 추가한다. 대시보드 작업은 사용자에게 안내하거나 확인을 받고 진행한다.
3. 본인만 쓰는 관리·모니터링 서비스는 Cloudflare Access로 보호한다. Proxmox UI와 SSH는 Tunnel로 공개하지 않는다.

## 진단

502/530은 cloudflared, 404는 Ingress host·Service 포트, 인증서 미발급은 `certificaterequest`·`challenge`와 HTTP-01 경로를 확인한다. 프록시 신뢰 설정은 IP 하드코딩 대신 Traefik 미들웨어로 처리한다.
