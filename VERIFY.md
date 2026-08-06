# 검증 방법

필요한 것은 OpenSSL 하나뿐입니다. 에렌델의 서버에 접속하거나 계정을 만들 필요가 없습니다.

## 1. 토큰의 서명 검증
openssl ts -verify
-in anchors/anchor_20260803.tsr
-queryfile anchors/anchor_20260803.tsq
-CAfile certs/cacert.pem
-untrusted certs/tsa.crt


`Verification: OK` 가 나오면 해당 토큰이 발급기관의 서명을 받은 진본입니다.

## 2. 봉인 대상 문서와 토큰이 일치하는지

sha256sum anchors/anchor_20260803.txt
openssl ts -reply -in anchors/anchor_20260803.tsr -text | grep -A1 "Message data"


두 값이 같으면, 그 문서가 **토큰에 기록된 시각에 이미 존재했다**는 뜻입니다.
문서를 한 글자라도 고치면 값이 달라지므로 사후 수정이 성립하지 않습니다.

## 3. 발급 시각 확인

openssl ts -reply -in anchors/anchor_20260803.tsr -text | grep -E "Time stamp|Serial number"


## 4. 소급 수정이 없었음을 확인

`INDEX.csv`의 `chain_digest` 열을 회차순으로 보십시오.

- 원장에 새 기록이 추가되지 않은 구간에서는 **값이 동일**해야 합니다
- 값이 바뀐 회차는 **기록 수(rows)도 함께 증가**해야 합니다

기록 수가 그대로인데 값만 바뀐 회차가 있다면 과거 기록이 수정된 것입니다.
현재까지 그런 회차는 없습니다.

## 5. 토큰 일련번호의 순서 확인

`INDEX.csv`의 `tsa_serial` 열을 보십시오. 발급 시각 순서와 일련번호 순서가 일치합니다.

이 번호는 발급기관이 전 세계 요청에 순차적으로 부여하므로 신청인이 선택할 수 없습니다.
특정 시점의 토큰을 나중에 만들어 끼워 넣으려면 그 시점의 번호 구간을 확보해야 하는데,
그 구간은 이미 다른 이용자의 토큰이 점유하고 있습니다.

## 6. 파일 무결성

sha256sum -c SHA256SUMS


## 검증에 실패했다면

ceo@earendel.kr 로 알려주십시오. 검증 실패는 저희에게 가장 중요한 정보입니다.
