# 검증 방법

필요한 것은 OpenSSL과 SHA-256 도구뿐입니다. 에렌델 서버나 계정은 필요하지 않습니다.

아래 예시는 `20260914` 회차를 확인합니다. 다른 회차는 날짜만 바꾸십시오.

```bash
anchor_date=20260914
```

## 1. 공개 파일의 SHA-256 확인

```bash
sha256sum -c SHA256SUMS
```

모든 항목이 `OK`이면 현재 내려받은 공개 파일이 `SHA256SUMS`와 일치합니다.

## 2. 시점 토큰과 요청 파일 확인

```bash
openssl ts -verify \
  -in "anchors/anchor_${anchor_date}.tsr" \
  -queryfile "anchors/anchor_${anchor_date}.tsq" \
  -CAfile certs/cacert.pem
```

`Verification: OK`가 나오면 토큰의 서명과 요청 파일의 메시지 지문이 일치합니다.

## 3. 봉인 대상 문서와 토큰 확인

```bash
openssl ts -verify \
  -in "anchors/anchor_${anchor_date}.tsr" \
  -data "anchors/anchor_${anchor_date}.txt" \
  -CAfile certs/cacert.pem
```

`Verification: OK`가 나오면 공개된 봉인 대상 문서의 SHA-256 지문이 토큰의 메시지 지문과 일치합니다. 따라서 해당 문서는 토큰에 기록된 시각까지 존재했으며, 이후 내용이 바뀌면 검증에 실패합니다.

## 4. TSA 발급 시각과 일련번호 확인

```bash
openssl ts -reply \
  -in "anchors/anchor_${anchor_date}.tsr" \
  -text |
grep -E "Serial number|Time stamp"
```

출력의 `Time stamp`가 RFC 3161 토큰 내부의 `genTime`입니다. `INDEX.csv`의 `tsa_gentime_utc` 값과 대조하십시오.

GitHub 커밋 시각은 공개 저장소에 게시된 시각입니다. TSA 발급 시각과 다를 수 있으며, 시점확인의 기준은 토큰의 `genTime`입니다.

## OpenSSL 경고 안내

`certs/tsa.crt`는 TSA의 최종 서명 인증서이며 CA 인증서가 아닙니다. 이 파일을 `-untrusted` 옵션에 넣으면 다음 경고가 발생할 수 있습니다.

```text
Warning: certificate ... is not a CA cert
```

이는 `-untrusted`가 중간 CA 인증서를 받는 옵션이기 때문입니다. 현재 공개된 `.tsr`에는 서명 인증서가 포함되어 있으므로 위 명령처럼 신뢰 루트인 `certs/cacert.pem`만 `-CAfile`로 지정하십시오. 이 방식으로 `Verification: OK`를 확인할 수 있습니다.

## 5. 원장 요약 변화 확인

`INDEX.csv`의 `chain_digest`와 `rows`를 회차순으로 대조하십시오.

- 원장에 새 기록이 추가되지 않은 구간에서는 `chain_digest`가 동일해야 합니다.
- `chain_digest`가 바뀐 회차는 `rows`의 변화와 함께 검토해야 합니다.
- 회차 날짜, TSA `genTime`, GitHub 게시 시각은 서로 다른 의미를 가질 수 있습니다.

## 검증에 실패했다면

실패한 회차, 실행한 명령, OpenSSL 버전과 오류 출력을 [ceo@earendel.kr](mailto:ceo@earendel.kr)로 알려주십시오.
