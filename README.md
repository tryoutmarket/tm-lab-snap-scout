# SNAP-SCOUT

레퍼런스 사진 한 장에서 답사 결정까지 — 스냅 사진 작가를 위한 로케이션 인텔리전스 도구.

🔗 **Live**: https://tryoutmarket.github.io/tm-lab-snap-scout/

## 1. 이게 뭔가
핀터레스트에 모아둔 레퍼런스 사진 앞에서 작가가 매번 마주하는 질문 — "여기 어디지? 지금도 이 모습일까? 거기서 정말 찍을 수 있을까?" — 에 대한 한 페이지짜리 답사 리포트를 생성한다.

## 2. 구현 형태
Claude Skill (웹앱 아님). `~/.claude/skills/snap-scout/` 에 설치되어 Claude Code에서 호출.

## 3. 한 장이 리포트가 되는 흐름
1. **IDENTIFY** — OCR + 비주얼 시그니처 + EXIF + 지도 검색으로 좌표 추정
2. **CONDITIONS** — 입장료·허가·골든아워·혼잡 시간대
3. **CURRENT** — 최신 데이터로 사진과 비교 (폐업·리모델링 감지)
4. **ALTERNATIVES** — 같은 무드의 3~5곳 대체 공간 추천

## 4. 컬렉션
- **№001** — 觀音堂 · 太平山街 (Hong Kong) → `02_sample-report.html`

## 5. 단계
v1 (단일 사진 리포트) → v2 (보드 일괄 분석) → v3 (작가 시그니처 추출) → v4 (큐레이션 결합).

---
© 2026 tryout-market / tm-lab
