# 견적 요청 폼 → 수입대행 견적서 매핑

영문 요약: The public form at `quote.html` posts to FormSubmit (`gtrade@mail.grokbot.com`) using the table email template. Prefer the hidden field `견적서_입력JSON` (`form_version` 1). Copy `paste` into the yellow input cells and ignore null or empty values. Everything under `context` is for the reply and for judging freight; it is not pasted into a calculated cell. A KRW figure is a budget note and must not be written to FOB.

양식 파일은 내부 견적 워크북 `수입대행_견적서_양식.xlsx`의 `견적서` 시트입니다. 고객이 아는 값만 받고, 환율·운임·보험·통관비·수수료·HS코드·관세율은 이 폼에 없습니다.

## 메일에서 읽는 법

1. 제목은 `[견적요청] {품목명}` 입니다. 품목명에 줄바꿈은 없습니다.
2. 본문은 FormSubmit `_template=table` 입니다. `name`이 곧 열 이름입니다.
3. `_subject`, `_template`, `_captcha`, `_next`, `_replyto`, `_honey`는 제어 필드입니다. 견적 값이 아닙니다.
4. `견적서_입력JSON`이 비어 있지 않으면 그 JSON을 기준으로 옮깁니다. 비어 있으면(스크립트 없는 제출) 아래 표의 필드 이름을 직접 읽습니다. 그때 `담당자 / 연락처`와 E24·F24 분리는 직접 해야 합니다.
5. 예시 숫자가 들어 있는 노란 셀은 워크북 사용법대로 지운 뒤 이번 요청만 입력합니다. 첫 품목 행은 24행입니다.

## JSON (`form_version`: 1)

`paste`의 값이 `null`이나 `""`이면 그 셀은 비웁니다.

| 키 | 의미 |
| --- | --- |
| `paste.C5` | 수신 (고객 상호) |
| `paste.C6` | 사업자등록번호 `000-00-00000`. 없으면 `""` |
| `paste.C7` | `담당자명 / 연락처`. 전화는 `010-0000-0000` 형식 |
| `paste.B24` | 품목명 |
| `paste.D24` | 수량 (정수) |
| `paste.E24` | `USD` 또는 `CNY`. 그 외는 `null` |
| `paste.F24` | 외화 FOB 단가. 통화가 USD/CNY일 때만 숫자 |

`context`는 셀에 붙이지 않습니다.

| 키 | 용도 |
| --- | --- |
| `이메일` | 견적 회신 주소. 시트에 이메일 칸 없음. Reply-To와 동일 |
| `모델·사양`, `참고 링크` | 품목을 특정. HS코드(C열)를 찾을 때 참고 |
| `통화`, `희망 단가` | 고객이 적은 원문 |
| `희망단가_KRW` | 통화가 KRW일 때의 희망 예산. **F24에 넣지 않음** |
| `희망 납기` | 납기 협의. 시트에 납기 칸 없음 |
| `배송지` | 시·도. 내륙운송비 F16(항구 → 고객 지정 장소)을 잡을 때 참고. F16에 지명을 적지 않음 |
| `요청사항` | 추가 품목은 여기에만 있을 수 있음. 두 번째부터는 B25 이하에 담당자가 나눠 적음 |
| `개인정보_수집이용_동의` | 접수 시점의 동의 값 `동의` |

## 화면 필드 → 시트

| 폼 `name` | 필수 | 시트 | 규칙 |
| --- | --- | --- | --- |
| `수신 (고객 상호)` | 예 | C5 | 그대로 |
| `사업자등록번호` | 아니오 | C6 | 10자리면 `000-00-00000` |
| `담당자명` | 예 | C7의 앞부분 | |
| `연락처` | 예 | C7의 뒷부분 | 휴대·지역·070 |
| `담당자 / 연락처` | 자동 | C7 | `{담당자명} / {연락처}` |
| `이메일` | 예 | 없음 | 회신·Reply-To |
| `품목명` | 예 | B24 | 대표 1건. 제목에도 사용 |
| `모델·사양` | 둘 중 하나 | 없음 | 참고 링크와 하나 이상 |
| `참고 링크` | 둘 중 하나 | 없음 | http(s) 주소. 1688·Alibaba 등 |
| `수량` | 예 | D24 | 1 이상의 정수 |
| `통화` | 단가 있을 때 | E24 | 드롭다운은 USD/CNY만. KRW는 E24에 넣지 않음 |
| `희망 단가` | 아니오 | F24 또는 참고 | USD/CNY → F24. KRW → `희망단가_KRW`만 |
| `희망 납기` | 예 | 없음 | 자유 입력 (월, 주, 협의) |
| `배송지` | 예 | F16 참고 | 시·도 이름만. 상세 주소 없음 |
| `요청사항` | 아니오 | 없음 | |
| `개인정보 수집·이용 동의` | 예 | 없음 | 값 `동의`. 수집 항목은 위 필드, 목적은 견적 회신, 보유는 1년 후 파기 |

고객이 넣지 않는 노란 셀: J5 견적일자, J6 유효기간, F10–F20 공통 조건, C24 HS코드, G24 관세율, 그리고 25행 이후. 폼은 금액을 계산하지 않고 결제도 받지 않습니다.

## 전송

- `POST https://formsubmit.co/gtrade@mail.grokbot.com`
- `_template=table`, `_captcha=true` (FormSubmit reCAPTCHA. 공개 접수라 켜 둠), `_honey` 비우면 통과
- `_next=https://greentrade.co.kr/quote-thanks.html`
- 이 주소로 처음 보내면 FormSubmit이 활성화 메일을 보냅니다. 그 링크를 누르기 전에는 견적 요청이 메일함에 쌓이지 않습니다. 활성화 메일을 만들려고 시험 제출하지 않습니다.
