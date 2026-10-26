# Sealed Secrets

컨트롤러는 `kube-system`에 있다. 커밋 대상은 `sealed-secret.yaml`, `secret.yaml.example`(플레이스홀더만), `pub-cert.pem`(공개 인증서)뿐이다.

## 시크릿 추가·변경

평문 원본은 Bitwarden `homelab` 폴더의 보안 메모(완성된 `secret.yaml` 전체)에 둔다. 값은 사용자가 직접 등록한다. 평문 파일을 만들지 않고 바로 암호화한다.

```bash
bw get notes <항목 id> | kubeseal --cert=pub-cert.pem --format yaml > applications/<앱>/sealed-secret.yaml
```

아직 Bitwarden에 없고 로컬 `secret.yaml`만 있으면 `kubeseal --cert=pub-cert.pem -f <secret.yaml> -w <sealed-secret.yaml> --format yaml`을 쓰고 Bitwarden 이전을 제안한다. 커밋 전 `git diff --cached --name-only`로 평문 `secret.yaml`이 스테이징되지 않았는지 확인한다.

시크릿 이름·네임스페이스는 암호화에 포함되므로 바꾸면 다시 암호화한다.

## 컨트롤러 키

컨트롤러는 30일마다 키를 추가하지만, 커밋된 시크릿은 모두 `pub-cert.pem`으로 암호화하므로 그 짝인 키 하나만 Bitwarden에 백업해 둔다(SKILL.md 항목 표). `pub-cert.pem`을 교체하면 짝인 키를 찾아 다시 백업하고 id를 갱신한다. 짝은 `openssl x509 -fingerprint -sha256`으로 `pub-cert.pem`과 각 키의 `tls.crt` 지문을 비교해 찾는다. 보안 메모는 암호화 후 10,000자 제한이 있으므로 메타데이터를 정리한 PEM `stringData` Secret 하나로 넣는다.

### 클러스터 재구축 후

- **키 백업 사용:** `bw get notes <키 항목 id> | ssh homelab kubectl --context default apply -f -` 후 컨트롤러를 재시작하고, 각 `SealedSecret`에서 `Secret`이 생성되는지 확인한다.
- **백업이 없을 때:** `kubeseal --fetch-cert --controller-namespace kube-system`으로 `pub-cert.pem`을 교체하고, `git ls-files '*sealed-secret.yaml'`의 모든 대상을 다시 암호화해 한 커밋으로 반영한 뒤 새 키를 백업한다.

`SealedSecret`이 복호화되지 않으면(`no key could decrypt secret`) 인증서 불일치나 이름·네임스페이스 변경을 의심한다. Secret 값은 디코딩해 출력하지 않는다.
