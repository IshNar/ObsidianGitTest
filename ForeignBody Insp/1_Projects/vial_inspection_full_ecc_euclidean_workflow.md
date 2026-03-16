# Vial 시약 검사 전체 순서 정리 (ECC + Euclidean 기준)

## 목적
회전/정지 과정에서 흔들리는 바이알 영상을 대상으로,

- **검사 대상이 아닌 영역**(뚜껑, 넥, 숄더, 바닥 곡면부, 불필요한 외곽부)을 제외하고
- **검사 유효 영역(Valid ROI)** 을 자동으로 추적/보정하며
- **정렬된 좌표계에서 차영상 기반 검사**를 수행하는 전체 구조를 정의한다.

본 문서는 특히 다음 상황을 전제로 한다.

- 바이알을 흡착 후 모터로 회전시킨다.
- 회전 중/정지 직후에는 진동, 미세 평행이동, 미세 회전이 존재할 수 있다.
- 검사 대상은 주로 **액체가 보이는 유효 영역 내부의 변화**이다.
- 검사 ROI는 수동 고정값이 아니라 **자동 정렬 결과를 따라가야 한다**.

---

## 핵심 원칙
1. **정렬용 기준**과 **검사용 ROI**는 분리한다.
2. 차영상이 크다고 해서 **정렬까지 중단하면 안 된다**.
3. 차영상 비교는 반드시 **정렬된 좌표계**에서 수행한다.
4. 검사 ROI는 사람이 매번 찍는 것이 아니라, **기준 ROI + 현재 pose(dx, dy, dθ)** 로 자동 갱신한다.
5. 정렬은 `ECC + Euclidean`으로 수행하고, 여기서 `dx`, `dy`, `dθ`를 함께 얻는다.

---

## 용어 정의
### 1) 정렬용 기준 이미지 (Reference Align Image)
현재 프레임을 위치 보정하기 위해 사용하는 마스터 이미지.

### 2) 정렬용 ROI (Alignment ROI)
ECC 정렬에 사용할 영역. 검사 목적이 아니라 **위치 보정 목적**이다.

권장 포함 영역:
- 바이알 외곽 실루엣
- 몸통 직선부 일부
- 캡/넥 경계 일부
- 숄더 형상 일부

권장 제외 영역:
- 액체 내부 패턴
- 액면 흔들림이 큰 구간
- 강한 반사 hotspot
- 움직이는 이물 가능 영역

### 3) 기준 검사 ROI (Reference Inspection ROI)
기준 좌표계에서 정의된 검사 허용 영역.
실제 런타임에서는 여기에 `dx, dy, dθ`를 적용해 현재 프레임용 검사 ROI를 생성한다.

### 4) 제외 마스크 (Exclude Mask)
검사하면 안 되는 영역.
예:
- 뚜껑 / 넥 / 숄더
- 바닥 곡면부
- 벽면 근처 margin
- 액면 근처
- 반복 반사 구역

### 5) Global Motion Score
이전/현재 프레임의 전역 변화량 지표.
이 값으로 “지금이 검사 가능한 타이밍인지”만 판단한다.
정렬 중단 여부를 결정하는 값이 아니다.

---

# 전체 검사 순서

## Step 1. 정렬용 기준 이미지 등록
정상 상태의 기준 이미지를 1장 이상 등록한다.

권장 조건:
- 초점/조명 조건이 안정적일 것
- 바이알 외곽/몸통/캡 등 구조물이 잘 보일 것
- 액체 내부 변화보다는 외형 기준이 명확할 것

주의:
- 액체 내부 무늬, 떠다니는 이물, 액면 흔들림은 정렬 기준으로 부적합하다.

---

## Step 2. 기준 ROI / 마스크 등록
기준 이미지 좌표계에서 아래 항목을 등록한다.

### 2-1. 정렬용 ROI 등록
ECC 정렬에 사용할 영역.

### 2-2. 기준 검사 ROI 등록
실제 검사 허용 영역.

### 2-3. 제외 마스크 등록
검사 금지 영역.

### 2-4. 필요 시 Zone Mask 등록
예:
- 중심부 검사 zone
- 벽면 근처 zone
- 액면 아래 zone

최종적으로는 보통 아래 구조를 사용한다.

```text
FinalValidMask = InspectionROI - ExcludeMask
```

---

## Step 3. 정렬용 표현 방식 결정
정렬용 기준 이미지와 현재 프레임을 어떤 표현으로 ECC에 넣을지 결정한다.

권장 우선순위:
1. **grayscale / normalized grayscale**
2. **gradient / edge 강조 영상**
3. **binary는 보조용**

### 권장 이유
- binary는 threshold 변화에 민감하다.
- 조명, 반사, 액체 상태 변화가 있으면 binary만으로는 흔들릴 수 있다.
- ECC는 일반적으로 grayscale 또는 gradient 기반이 더 안정적인 경우가 많다.

실무 권장:
- coarse 검출: binary/contour 사용 가능
- fine 정렬(ECC): grayscale 또는 gradient 사용 권장

---

## Step 4. Continuous Grab 수행
카메라로 프레임을 연속 취득한다.

각 프레임마다 최소한 아래 데이터를 관리한다.
- 현재 원본 프레임
- 직전 프레임
- 현재 pose 추정값
- 정렬 성공 여부
- Global Motion Score

---

## Step 4-1. 프레임 간 전역 변화량(Global Motion Score) 계산
직전 프레임과 현재 프레임의 변화량을 계산한다.

예시:
```text
G(t) = sum(abs(I_t - I_{t-1})) / ROI_area
```
또는
```text
G(t) = changed_pixel_count / ROI_area
```

이 단계의 목적은 아래 둘 중 하나를 판별하는 것이다.
- 아직 액체 전체 유동이 큰 상태인가?
- 검사 가능한 안정 구간에 진입했는가?

### 중요
`G(t)`가 크다고 해서 **정렬 자체를 건너뛰면 안 된다.**

정확한 운용은 아래와 같다.
- `G(t)`가 너무 크면 → **정밀 검사 skip**
- 그래도 → **거친 추적/정렬은 계속 유지**

즉,

```text
Global motion이 큼 = 검사 보류
Global motion이 큼 ≠ 위치 추적 중단
```

---

## Step 5. 차영상이 과도하지 않을 때 ECC + Euclidean으로 정렬
Global Motion Score가 허용 범위 이하로 내려오면,
정렬용 기준 이미지와 현재 프레임을 `ECC + Euclidean`으로 정렬한다.

### 목적
현재 프레임의 pose를 기준 좌표계에 맞게 추정한다.

### 사용 개념
OpenCV 기준:
- `findTransformECC(..., MOTION_EUCLIDEAN)`

Euclidean 모델은 보통 아래 변환을 의미한다.
- 회전 `dθ`
- 평행이동 `dx`, `dy`

즉 Step 5에서는 별도의 패턴 매칭 peak 탐색 대신,
**ECC 최적화 결과 warp matrix 자체에서 `dx`, `dy`, `dθ`를 얻는 구조**로 간다.

### 결과 해석
ECC 결과 2x3 warp matrix를 예로 들면 보통 다음과 비슷한 형태가 된다.

```text
[ a  b  tx ]
[ -b a  ty ]
```

여기서 일반적으로:
- `tx` → `dx`
- `ty` → `dy`
- `θ = atan2(b, a)` → `dθ`

다만 실제 부호(sign)는 아래 조건에 따라 달라질 수 있다.
- reference → current 로 해석하는지
- current → reference 로 해석하는지
- `WARP_INVERSE_MAP` 사용 여부

따라서 구현 시에는 **warp 적용 방향과 dθ 부호를 실제 테스트로 검증해야 한다.**

### 정렬 실패 처리
ECC 점수 또는 반복 종료 상태가 불안정하면:
- 현재 프레임 검사 skip
- 이전 pose 유지 또는 fallback pose 사용
- 필요 시 coarse alignment 재시도

---

## Step 6. 기준 ROI / Exclude Mask / Zone Mask에 pose(dx, dy, dθ) 적용
Step 5에서 얻은 pose를 기준 좌표계의 마스크들에 적용하여,
현재 프레임용 마스크를 생성한다.

변환 대상:
- 기준 검사 ROI
- Exclude Mask
- Zone Mask

즉:

```text
CurrentInspectionROI = Warp(ReferenceInspectionROI, dx, dy, dθ)
CurrentExcludeMask   = Warp(ReferenceExcludeMask, dx, dy, dθ)
CurrentZoneMask      = Warp(ReferenceZoneMask, dx, dy, dθ)
```

그리고 최종 검사 허용 마스크는:

```text
CurrentFinalValidMask = CurrentInspectionROI - CurrentExcludeMask
```

### 중요
사람이 매 프레임 ROI를 다시 찍는 구조가 아니다.
**기준 ROI를 현재 pose에 따라 자동 갱신하는 구조**이다.

---

## Step 7. 차영상 비교 전, 비교 대상 프레임도 동일 좌표계로 정렬
검사에 사용할 프레임들(예: 이전/현재, 현재/다음)도 같은 기준 좌표계로 맞춘다.

즉:
- `I(t-1)` → align
- `I(t)` → align
- 필요 시 `I(t+1)` → align

그 후 차영상을 계산한다.

### 매우 중요
다음 방식은 위험하다.

```text
abs(raw_current - raw_prev)
```

이유:
- 진동
- 미세 이동
- 미세 회전
- 광학 흔들림

때문에 가짜 차영상이 발생할 수 있다.

반드시 아래처럼 해야 한다.

```text
D = abs(Aligned(I_t) - Aligned(I_{t-1}))
```

---

## Step 8. 정렬된 프레임 기준으로 차영상 생성
정렬된 프레임끼리 차영상을 계산한다.

예:
```text
D1 = abs(Aligned(I_t) - Aligned(I_{t-1}))
D2 = abs(Aligned(I_{t+1}) - Aligned(I_t))
```

필요 시:
- 2프레임 차분
- 3프레임 차분
- 누적 차분
- 최대값 누적

등을 사용할 수 있다.

---

## Step 9. 차영상이 유효 검사 구간(sweet spot)인지 판정
차영상이 너무 크거나 너무 작으면 바로 검사하지 않는다.

### 9-1. 차영상이 너무 큰 경우
의미:
- 아직 액체 전체 유동이 큼
- bulk motion이 남아 있음

처리:
- 현재 프레임 검사 skip
- 다음 프레임으로 진행

### 9-2. 차영상이 너무 작은 경우
의미:
- 거의 정지 상태
- 정보량이 부족할 수 있음

처리:
- 검사 종료 또는 low confidence 처리

### 9-3. 차영상이 중간 범위인 경우
의미:
- 전역 유동은 줄었고
- 국소 움직임 또는 유효 변화만 남기 쉬운 구간

처리:
- 이 구간에서만 상세 blob 검사 수행

즉:

```text
Too High  -> skip
Sweet Spot -> inspect
Too Low   -> stop or low confidence
```

---

## Step 10. CurrentFinalValidMask 내부에서만 검사 수행
차영상 전체를 바로 검사하지 않고,
반드시 현재 프레임용 유효 마스크 내부에서만 후보를 받는다.

즉:

```text
ValidDiff = DiffImage AND CurrentFinalValidMask
```

이 단계의 목적:
- 뚜껑 검사 방지
- 넥/숄더 검사 방지
- 외부 불필요 영역 검사 방지
- 벽면 margin 제외
- 액면 근처 제외

---

## Step 11. blob 분석 및 최종 판정
유효 마스크 내부의 차영상 후보에 대해 blob 분석을 수행한다.

예시 feature:
- 면적
- width / height
- 장축 / 단축비
- 원형도
- 평균 밝기 변화
- 중심점 위치
- 프레임 지속성

추가 권장 조건:
- blob 중심점이 유효 마스크 내부에 있을 것
- blob 면적의 일정 비율 이상이 유효 마스크 내부에 있을 것
- 제외 경계로부터 최소 거리 이상 떨어질 것

---

# 전체 순서 한 줄 요약

```text
기준 이미지/ROI 등록
-> Continuous Grab
-> Global Motion Score 계산
-> 검사 가능한 구간이면 ECC + Euclidean 정렬
-> dx, dy, dθ 획득
-> 기준 ROI/Mask를 현재 위치로 변환
-> 비교 프레임도 동일 좌표계로 정렬
-> 정렬된 차영상 생성
-> FinalValidMask 내부에서만 blob 검사
-> 최종 판정
```

---

# 구현 시 꼭 지켜야 할 수정 포인트

## 1. 차영상이 크다고 정렬까지 건너뛰지 말 것
틀린 구조:
```text
if (diff_big) continue;
```

권장 구조:
```text
if (diff_big) {
    // inspection skip
    // but coarse tracking / pose update maintain
}
```

## 2. dx, dy, dθ는 ECC + Euclidean 결과에서 함께 얻을 것
이번 보정본에서는 Step 5를 패턴 매칭 기반이 아니라
**ECC + Euclidean warp 결과 기반 pose 추정**으로 통일한다.

## 3. 차영상은 raw frame끼리 빼지 말 것
반드시 정렬 후 비교:
```text
abs(Aligned(I_t) - Aligned(I_{t-1}))
```

---

# OpenCV 구현 관점 메모

## ECC 관련
- `findTransformECC()` 사용
- motion type: `MOTION_EUCLIDEAN`
- 초기 warp는 identity로 시작 가능
- 이전 프레임의 warp를 초기값으로 재사용하면 수렴 안정성에 도움이 될 수 있음

## 프레임 전처리 권장
- grayscale 변환
- 필요 시 Gaussian blur 소량
- histogram normalization 또는 CLAHE 검토
- 강한 반사/액면 구간은 alignment ROI에서 제외 권장

## 마스크 적용
ECC 정렬용 mask와 검사 mask는 분리 가능하다.
- ECC mask: 정렬 안정성 확보용
- Inspection mask: 검사 허용 영역 제한용

---

# Antigravity 전달용 요구사항 요약
아래 내용을 기준으로 수정/구현 요청.

```text
다음 구조로 바이알 검사 로직을 정리/구현해줘.

1. 정렬용 기준 이미지, 정렬용 ROI, 기준 검사 ROI, 제외 마스크를 등록한다.
2. Continuous Grab으로 프레임을 계속 취득한다.
3. 프레임 간 Global Motion Score를 계산한다.
4. Global Motion Score가 너무 크면 정밀 검사는 skip하되, 위치 추적/정렬 상태는 유지한다.
5. Global Motion Score가 허용 범위로 들어오면, 정렬용 기준 이미지와 현재 프레임을 OpenCV ECC + MOTION_EUCLIDEAN으로 정렬한다.
6. ECC 결과 warp matrix에서 dx, dy, dθ를 추출한다.
7. 기준 검사 ROI / 제외 마스크 / Zone Mask에 dx, dy, dθ를 적용하여 현재 프레임용 마스크를 생성한다.
8. 이전/현재(필요 시 다음) 프레임도 동일 기준 좌표계로 warp하여 정렬한다.
9. 정렬된 프레임끼리 차영상을 생성한다. raw frame끼리 직접 차감하지 않는다.
10. 차영상이 sweet spot 범위일 때만 CurrentFinalValidMask 내부에서 blob 검사를 수행한다.
11. blob 중심점, 면적 겹침율, 크기, 지속성 등으로 최종 판정한다.

주의사항:
- dx, dy, dθ는 template matching이 아니라 ECC + Euclidean 결과에서 얻는다.
- 차영상이 크다고 해서 정렬까지 중단하면 안 된다.
- 검사 ROI는 사람이 매 프레임 수동 지정하는 것이 아니라, 기준 ROI를 현재 pose(dx, dy, dθ)에 따라 자동 변환하는 구조여야 한다.
- 뚜껑, 넥, 숄더, 액면, 벽면 margin, 바닥부 등은 제외 마스크로 관리하고 검사 대상에서 제외한다.
```

---

# 최종 결론
이번 보정본의 핵심은 아래 3가지다.

1. **Step 5는 ECC + Euclidean으로 통일**한다.
2. `dx`, `dy`, `dθ`는 ECC 결과 warp에서 함께 얻는다.
3. **전체 검사 로직은 “정렬 -> ROI 자동 보정 -> 정렬된 차영상 검사” 순서**로 간다.
