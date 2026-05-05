# 10분 안에 판단하라 — 프로젝트 전체 계획

> **프로젝트**: 재난 대피 강의 웹 프레젠테이션 + 나레이션 음성 시스템
> **대상**: 대학생 / 일반 시민
> **최종 목표**: 웹 슬라이드에서 버튼 하나로 2인 나레이션 음성이 자동 재생되는 완성본
> **최종 업데이트**: 2026-05-05

---

## Phase 0 — 기획 및 콘텐츠 설계 ✅ 완료

### 0-1. 주제 및 구성 기획
- 주제 확정: 화재·지진 10분 생존 판단법
- 대상 청중: 대학생 (v1), 일반 시민 (v2)
- 슬라이드 수: 27슬라이드 (4파트 + 표지/목차/결론)

### 0-2. 콘텐츠 조사 및 검증
- 참고 자료: 행정안전부, 소방청, 기상청, KOSHA 공식 자료
- 수치 근거 확인: 산소농도별 반응, 연기 확산 속도, 골든타임, 내진설계 기준 변천

### 0-3. 발표 대본 작성 ✅
- 파일: `results/나레이션_대본.md`
- 형식: 발표자 문종욱 1인 기준 단독 발표 스크립트
- 슬라이드 1~27 전체 커버
- 예상 발표 시간: 약 25~30분

---

## Phase 1 — 웹 프레젠테이션 제작 ✅ 완료

### 1-1. HTML 슬라이드 제작
- 파일: `results/10분_안에_판단하라_최종.html`
- 슬라이드: 27개 (`slide-0` ~ `slide-26`)
- 기능: 키보드/터치 네비게이션, 프래그먼트 애니메이션, 진행바, 풀스크린

### 1-2. 버전 관리
| 파일 | 내용 |
|---|---|
| `10분_안에_판단하라_v2_6_1.html` | 중간 버전 |
| `10분_안에_판단하라_github.html` | GitHub Pages 배포용 |
| `10분_안에_판단하라_최종.html` | 현재 메인 버전 |

### 1-3. PDF 출력
| 파일 | 대상 |
|---|---|
| `10분 안에 판단하라 - 대학생을 위한 재난 대피 강의_4.pdf` | 대학생용 |
| `10분_안에_판단하라(일반시민용).pdf` | 일반 시민용 |

---

## Phase 2 — 티키타카 나레이션 시스템 ✅ 대본 확정

### 2-1. 성인용 대본 ✅ 확정 (v1.3)
- 파일: `results/티키타카_대본.md`
- 화자: 지수(여성 진행자) + 종욱(남성 전문가)
- 음성 설정: `ko-KR-SunHiNeural` (지수, 표준) / `ko-KR-InJoonNeural` (종욱, 표준)
- 슬라이드 27개 전체 커버

### 2-2. 대학생용 대본 ✅ 확정 (v1.0)
- 파일: `results/티키타카_대본_대학생용.md`
- 화자: 지수(여2학년) + 민준(남3학년 선배)
- 음성 설정: `ko-KR-SunHiNeural` (지수, rate +10%, pitch +8Hz — 발랄한 톤)
             `ko-KR-InJoonNeural` (민준, rate +8% — 친근한 선배 톤)
- 슬라이드 27개 전체 커버

### 2-3. ElevenLabs TTS 엔진 (기존 최종.html) ✅ 구현됨
- 방식: ElevenLabs API (`eleven_multilingual_v2`)
- 음성: 지수 (`ETPP7D0aZVdEj12Aa7ho`) / 종욱 (`ZJCNdZEjYwkOElxugmW2`)
- 폴백: Web Speech API
- ⚠️ API 키 하드코딩 보안 이슈 — Phase 3 MP3 방식으로 대체 예정

---

## Phase 3 — edge-tts MP3 사전 생성 + HTML 2버전 완성 🔄 진행 예정

> ElevenLabs 실시간 API 대신 edge-tts로 MP3를 미리 생성해 HTML에 임베드.
> API 키 불필요, 인터넷 없이 재생 가능, 2가지 톤(성인용/대학생용) 분리 적용.

### 3-1. 의존성 설치
```bash
pip install edge-tts mutagen --break-system-packages
```
- edge-tts: Microsoft TTS (무료, 한국어 Neural 지원)
- mutagen: MP3 메타데이터·길이 추출

### 3-2. narration-segments.json 2종 작성
- `narration-segments-adult.json` — 성인용 (지수+종욱, 표준 톤)
- `narration-segments-student.json` — 대학생용 (지수+민준, 발랄한 톤)
- 구조: 슬라이드 단위, 화자(f/m), 대사 텍스트

### 3-3. edge-tts 음성 프로파일

| 버전 | 화자 | 목소리 | rate | pitch | 특징 |
|---|---|---|---|---|---|
| 성인용 | 지수 | `ko-KR-SunHiNeural` | 기본 | 기본 | 차분하고 전문적 |
| 성인용 | 종욱 | `ko-KR-InJoonNeural` | 기본 | 기본 | 안정적인 전문가 |
| 대학생용 | 지수 | `ko-KR-SunHiNeural` | +10% | +8Hz | 밝고 발랄한 여학생 |
| 대학생용 | 민준 | `ko-KR-InJoonNeural` | +8% | +5Hz | 친근한 남학생 선배 |

### 3-4. TTS 생성 스크립트 실행
```bash
# 성인용
python scripts/generate-segment-tts.py narration-segments-adult.json --out audio/adult/

# 대학생용 (rate/pitch 파라미터 포함)
python scripts/generate-segment-tts.py narration-segments-student.json --out audio/student/ \
  --rate-f "+10%" --pitch-f "+8Hz" --rate-m "+8%" --pitch-m "+5Hz"
```
- 출력: `audio/adult/s{N}-seg-*.mp3` + `audio/adult/s{N}-timeline.json`
         `audio/student/s{N}-seg-*.mp3` + `audio/student/s{N}-timeline.json`

### 3-5. HTML 2버전 완성
- `10분_안에_판단하라_최종.html` — 성인용 MP3 오디오 엔진 교체 (ElevenLabs 제거)
- `10분_안에_판단하라_대학생용.html` — 대학생 talkScript + 대학생용 MP3 신규 생성
- 공통 기능: 슬라이드 전환 시 해당 슬라이드 대사 자동 재생 / ⏸ 일시정지 / 🔊 ON/OFF

### 3-6. 검증
- 전체 27슬라이드 재생 확인
- 화자 전환 타이밍 확인 (지수→민준/종욱 자연스럽게 이어지는지)
- 대학생용 톤이 충분히 발랄하게 들리는지 청취 확인 → 필요 시 rate/pitch 재조정

---

## Phase 4 — 영상 렌더링 ⬜ 장기 계획

> AutoPresent Studio Phase 1-3 구현 완료 후 진행 가능.

### 4-1. Remotion 프로젝트 초기화
- `video/` 폴더에 Remotion v4 설정
- 슬라이드 컴포넌트 연결

### 4-2. 오디오 타이밍 동기화
- `durations.json` 생성 (각 슬라이드 MP3 재생 시간)
- Remotion `<Series>` + `<Audio>` 조합

### 4-3. 자막 (선택)
- `scripts/generate-soft-subtitles.py` → `.srt` 생성
- burn-in 또는 soft subtitle 선택

### 4-4. 최종 MP4 렌더링
```bash
cd video && npx remotion render
```
- 출력: `output.mp4` (1920×1080, 15fps)

---

## 현재 파일 구조 (results/)

```
results/
├── 나레이션_대본.md              ← 1인 발표 스크립트 (문종욱 단독)
├── 티키타카_대본.md              ← 2인 대화 스크립트 (지수 + 종욱) ★ 이 파일
├── plan.md                      ← 프로젝트 전체 계획 (이 파일)
├── 10분_안에_판단하라_최종.html  ← 현재 메인 웹 프레젠테이션
├── 10분_안에_판단하라_v2_6_1.html
├── 10분_안에_판단하라_github.html
├── 10분 안에 판단하라 - 대학생을 위한 재난 대피 강의_4.pdf
└── 10분_안에_판단하라(일반시민용).pdf
```

---

## 대본 업데이트 규칙

대본 수정이 발생할 때마다:
1. `티키타카_대본.md` 해당 슬라이드 수정
2. `10분_안에_판단하라_최종.html` 내 `talkScript` 배열 동기화
3. `티키타카_대본.md` 하단 변경 이력 테이블 업데이트

---

## 마일스톤 요약

| 단계 | 내용 | 상태 |
|---|---|---|
| Phase 0 | 기획 + 발표 대본 | ✅ 완료 |
| Phase 1 | HTML 웹 프레젠테이션 | ✅ 완료 |
| Phase 2-1 | 성인용 티키타카 대본 v1.3 확정 | ✅ 완료 |
| Phase 2-2 | 대학생용 티키타카 대본 v1.0 확정 | ✅ 완료 |
| Phase 2-3 | ElevenLabs TTS 엔진 (성인용 HTML) | ✅ 구현됨 (API 키 보안 이슈 있음) |
| Phase 3-1 | edge-tts 설치 + segments JSON 2종 작성 | 🔄 다음 작업 |
| Phase 3-2 | 성인용 MP3 생성 (표준 톤) | ⬜ 예정 |
| Phase 3-3 | 대학생용 MP3 생성 (발랄한 톤) | ⬜ 예정 |
| Phase 3-4 | 성인용 HTML 오디오 엔진 교체 | ⬜ 예정 |
| Phase 3-5 | 대학생용 HTML 신규 생성 | ⬜ 예정 |
| Phase 4 | Remotion MP4 영상 렌더링 | ⬜ 장기 |
