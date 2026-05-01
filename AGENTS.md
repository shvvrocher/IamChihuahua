# IamChihuahua

웹캠으로 사람 얼굴을 인식해서 실시간으로 치와와 얼굴 필터를 씌워주는 웹앱.

## 실행 방법

```bash
python3 -m http.server 8080
# 브라우저에서 http://localhost:8080 접속
```

`file://`로 직접 열면 CORS 오류로 MediaPipe 모델 로딩이 막히므로 반드시 로컬 서버를 통해 열어야 한다.

## 파일 구성

```
index.html       # 앱 전체 (HTML/CSS/JS 단일 파일)
Chihuahua.webp   # 치와와 참고 이미지 (현재 미사용, 텍스처 확장용)
AGENTS.md        # 이 파일
```

## 기술 스택

- **MediaPipe FaceLandmarker** (CDN) — 468개 랜드마크 실시간 추적, VIDEO 모드
- **Canvas 2D API** — 필터 렌더링 (별도 라이브러리 없음)
- 순수 HTML/CSS/JavaScript 단일 파일, 서버 불필요, GH Pages 배포 가능

## 필터 구조 (`index.html` 내 주요 함수)

| 함수 | 역할 |
|---|---|
| `drawFilter(landmarks)` | 매 프레임 필터 전체 진입점 |
| `drawEar(tipX, tipY, w, h, lean, flip)` | 치와와 귀 (좌/우 각각 호출) |
| `drawTeeth(lipBox, mouthTop, mouthBot, openness)` | 이빨 (입 벌릴 때만 표시) |
| `htDots(clipPath, x0, y0, x1, y1, spacing, densityFn)` | 망점 헬퍼 — 헥사고날 그리드로 점 찍기 |
| `renderLoop()` | rAF 기반 프레임 루프, 거울 모드(좌우 반전) 적용 |

## 랜드마크 좌표계

MediaPipe 반환값은 0~1 정규화 좌표 → `lm()` / `bbox()` 헬퍼로 캔버스 픽셀 좌표로 변환.  
거울 모드이므로 x 좌표를 `1 - x`로 반전 후 사용.

## 주요 상수

- `HT = '#1515c8'` — 실크스크린 블루 (망점 전용 컬러)
- `EYE_SCALE = 2.3` — 눈 확대 배율
- 귀 위치 기준: 랜드마크 `#10` (정수리)

## 배포

GitHub Pages: `index.html`, `Chihuahua.webp`를 같은 경로에 두면 바로 동작.
