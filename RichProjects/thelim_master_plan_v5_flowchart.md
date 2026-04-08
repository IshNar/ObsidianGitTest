# TheLim 마스터 플랜 v5 — Flow 차트

> 상단: 전체 조감도 (1장)
> 하단: 계층별 상세 (5개 다이어그램)
>
> 7일 생활 패턴 전체를 이 문서 하나로 커버.

---

## 0. 전체 조감도 (한 장)

```mermaid
flowchart TB
    Start([매일 아침 6:30 기상]) --> Check{오늘 컨디션?}

    Check -->|"FULL<br/>어제 7h+ 수면<br/>정시 퇴근 예상"| Full[FULL 모드]
    Check -->|"REDUCED<br/>어제 5~6h 수면<br/>야근·회의 폭주"| Reduced[REDUCED 모드]
    Check -->|"MINIMUM<br/>어제 5h 이하<br/>컨디션 60% 이하"| Minimum[MINIMUM 모드<br/>주 2회 한도]

    Full --> DayType{요일?}
    Reduced --> DayType
    Minimum --> DayType

    DayType -->|월~금| Weekday[평일 루틴]
    DayType -->|토| Saturday[토요일<br/>모의고사+긴 유산소]
    DayType -->|일| Sunday[일요일<br/>주간 리뷰+회복]

    Weekday --> Morning[아침 TOEFL 60분<br/>단어+리스닝+로테이션]
    Morning --> Work[9~18시 회사 업무<br/>+점심 모바일 지시]
    Work --> Evening[저녁 TOEFL 35분<br/>스피킹or라이팅]
    Evening --> PC{PC 집중 시간<br/>19:35~21:00}

    PC -->|"4/11~4/24<br/>2주 집중"| Bridge[AI_CONNECT_BRIDGE<br/>실사용+피드백]
    PC -->|"4/25~"| Rotation[요일별 로테이션<br/>월:회사 화:GameTrade<br/>수:SmartStock 목:Parallel<br/>금:GamePromo]

    Bridge --> Night[저녁 운동 30분<br/>+OMSCS 1h<br/>+chin tuck]
    Rotation --> Night
    Saturday --> SatNight[저녁 자유]
    Sunday --> SunNight[다음주 계획]
    Night --> Sleep([23:30 취침<br/>7h 수면])
    SatNight --> Sleep
    SunNight --> Sleep

    Sleep -.->|다음 날| Start

    subgraph Gates[게이트 · 불변]
        G1[건강 최우선<br/>식단·운동·수면]
        G2[TOEFL 매일 1h35m<br/>절대 스킵 금지]
        G3[6월 중순 시험]
        G4[4/24 AI_CONNECT_BRIDGE<br/>2주 실사용 종료]
    end

    subgraph Frozen[4/11~4/24 동결]
        F1[GameTrade]
        F2[GamePromo]
        F3[ParallelPlay]
        F4[InspectNote]
        F5[Smart Stock]
        F6[Voca Project]
        F7[Lite Inspect]
    end

    classDef fullMode fill:#d4edda,stroke:#28a745,color:#000
    classDef reducedMode fill:#fff3cd,stroke:#ffc107,color:#000
    classDef minMode fill:#f8d7da,stroke:#dc3545,color:#000
    classDef gate fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef frozen fill:#e2e3e5,stroke:#6c757d,color:#000

    class Full fullMode
    class Reduced reducedMode
    class Minimum minMode
    class G1,G2,G3,G4 gate
    class F1,F2,F3,F4,F5,F6,F7 frozen
```

---

## 1. 컨디션 판정 & 모드 분기 상세

```mermaid
flowchart TB
    Wake([6:30 기상]) --> Q1{어제 수면시간?}

    Q1 -->|7시간 이상| Q2{오늘 일정?}
    Q1 -->|5~6시간| R1[REDUCED 후보]
    Q1 -->|5시간 이하| M1[MINIMUM 후보]

    Q2 -->|정시 퇴근 가능| Q3{신체 신호?}
    Q2 -->|야근·회의 폭주 예상| R1

    Q3 -->|두통·피로·소화불량 X| FULL[FULL 모드 ✅]
    Q3 -->|가벼운 피로만| FULL
    Q3 -->|신호 있음| R1

    R1 --> R2{MINIMUM 사용 횟수<br/>이번 주}
    M1 --> R2

    R2 -->|0~1회| MIN[MINIMUM 모드 ✅<br/>회복 우선]
    R2 -->|2회 도달| RED[REDUCED 모드 ✅<br/>더 이상 MIN 불가]

    FULL --> FullDetail[아침 운동 20분<br/>TOEFL 1h35m<br/>PC 85분<br/>OMSCS 1h<br/>저녁 운동 30분<br/>23:30 취침]

    RED --> RedDetail[아침 스트레칭 10분<br/>TOEFL 1h35m 그대로<br/>PC 30분 or 스킵<br/>OMSCS 30분<br/>저녁 운동 15분<br/>23:00 취침 ↑]

    MIN --> MinDetail[운동 스킵 OK<br/>TOEFL 단어15m+리스닝15m만<br/>저녁 35분 스킵 OK<br/>식단 룰 유지<br/>chin tuck 5분<br/>22:00 취침]

    classDef fullMode fill:#d4edda,stroke:#28a745,color:#000
    classDef reducedMode fill:#fff3cd,stroke:#ffc107,color:#000
    classDef minMode fill:#f8d7da,stroke:#dc3545,color:#000
    classDef absolute fill:#cfe2ff,stroke:#0d6efd,color:#000

    class FULL,FullDetail fullMode
    class RED,RedDetail reducedMode
    class MIN,MinDetail minMode
```

### MINIMUM 모드에서도 절대 스킵 금지

```mermaid
flowchart LR
    MinMode([MINIMUM 모드]) --> Never1[TOEFL 단어 15분]
    MinMode --> Never2[TOEFL 리스닝 15분<br/>출근 중 이어폰]
    MinMode --> Never3[저녁 식단 룰<br/>단백질+채소<br/>야식 X]
    MinMode --> Never4[chin tuck 5분<br/>취침 전]

    Never1 --> Done([최소 완료])
    Never2 --> Done
    Never3 --> Done
    Never4 --> Done

    classDef minMode fill:#f8d7da,stroke:#dc3545,color:#000
    classDef mandatory fill:#cfe2ff,stroke:#0d6efd,stroke-width:2px,color:#000

    class MinMode minMode
    class Never1,Never2,Never3,Never4 mandatory
```

---

## 2. 평일 FULL 모드 타임라인

```mermaid
flowchart TB
    T0630[6:30~7:00<br/>공복 유산소 20분<br/>걷기/실내자전거] --> T0700
    T0700[7:00~7:20<br/>아침 황제 식사<br/>볶음 덮밥<br/>현미밥 1공기+계란 3개<br/>+참치 반캔+채소+김<br/>~630kcal 단백40g] --> T0720
    T0720[7:20~7:35<br/>TOEFL 단어 15분<br/>26초 토단정 30단어] --> T0735
    T0735[7:35~7:50<br/>TOEFL 리스닝 15분<br/>Howard Macs+쉐도잉] --> T0750
    T0750[7:50~8:20<br/>TOEFL 로테이션 30분<br/>월:리딩 화:리스닝<br/>수:리딩 목:리스닝<br/>금:리딩] --> T0820
    T0820[8:20~9:00<br/>출근 중 모바일<br/>AI 작업 결과 리뷰<br/>Discord+Slack] --> T0900

    T0900[9:00~12:00<br/>회사 업무] --> T1200
    T1200[12:00~13:00<br/>점심 귀족 식사<br/>회사 샐러드<br/>잎채소+닭가슴살 100g<br/>+올리브오일 드레싱<br/>~500kcal 단백30g<br/>+모바일 프로젝트 지시] --> T1300
    T1300[13:00~17:30<br/>회사 업무<br/>+Codex 코드 검토] --> T1730
    T1730[17:30~18:00<br/>퇴근 준비<br/>AI 결과 정리] --> T1800

    T1800[18:00~18:30<br/>퇴근 중 모바일<br/>AI 작업 지시] --> T1830
    T1830[18:30~19:00<br/>저녁 빈자 식사<br/>삶은 계란 2~3개<br/>+WPI 1스쿱 물 250ml<br/>+생채소<br/>~400kcal 단백44g<br/>19시 이전 종료] --> T1900
    T1900[19:00~19:30<br/>TOEFL 로테이션 30분<br/>월수금:스피킹<br/>화목:라이팅] --> T1930
    T1930[19:30~19:35<br/>TOEFL 발음 5분<br/>th·r/l + TED Talk] --> T1935
    T1935[19:35~21:00<br/>PC 집중 85분<br/>2주 모드 or 일반 모드<br/>분기는 섹션 4 참조] --> T2100
    T2100[21:00~21:30<br/>저녁 운동 30분<br/>요일별 근력/유산소] --> T2130
    T2130[21:30~21:35<br/>WPI 추가 1스쿱<br/>물 250ml<br/>근합성 골든타임<br/>월수금 근력일만<br/>저녁 것과 별개] --> T2135
    T2135[21:35~22:30<br/>OMSCS 1h<br/>수학/딥러닝] --> T2230
    T2230[22:30~22:55<br/>AI 밤새 작업 지시<br/>내일 할 일 정리] --> T2255
    T2255[22:55~23:00<br/>chin tuck 5분<br/>경추 케어] --> T2300
    T2300([23:00~23:30<br/>마무리+취침<br/>7h 수면])

    subgraph MorningSection[오전 6:30~9:00]
        T0630
        T0700
        T0720
        T0735
        T0750
        T0820
    end

    subgraph WorkSection[낮 9:00~18:00 · 회사 업무]
        T0900
        T1200
        T1300
        T1730
    end

    subgraph EveningSection[저녁 18:00~23:30]
        T1800
        T1830
        T1900
        T1930
        T1935
        T2100
        T2130
        T2135
        T2230
        T2255
        T2300
    end

    classDef toefl fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef exercise fill:#d4edda,stroke:#28a745,color:#000
    classDef food fill:#fff3cd,stroke:#ffc107,color:#000
    classDef work fill:#e2e3e5,stroke:#6c757d,color:#000
    classDef project fill:#f8d7da,stroke:#dc3545,color:#000

    class T0720,T0735,T0750,T1900,T1930 toefl
    class T0630,T2100,T2255 exercise
    class T0700,T1200,T1830,T2130 food
    class T0900,T1300,T1730 work
    class T0820,T1800,T1935,T2135,T2230 project
```

---

## 3. REDUCED · MINIMUM 모드 축소 규칙 (항목별)

### 3-1. 아침 운동

```mermaid
flowchart TB
    A([아침 운동]) --> AF[FULL<br/>공복 유산소 20분<br/>걷기/실내자전거]
    AF --> AR[REDUCED<br/>스트레칭 10분<br/>가벼운 걷기]
    AR --> AM[MINIMUM<br/>스킵 OK]

    classDef fullMode fill:#d4edda,stroke:#28a745,color:#000
    classDef reducedMode fill:#fff3cd,stroke:#ffc107,color:#000
    classDef minMode fill:#f8d7da,stroke:#dc3545,color:#000
    class AF fullMode
    class AR reducedMode
    class AM minMode
```

### 3-2. TOEFL

```mermaid
flowchart TB
    T([TOEFL]) --> TF[FULL<br/>아침 60분 + 저녁 35분<br/>= 1h 35m 풀 실행]
    TF --> TR[REDUCED<br/>1h 35m 그대로 유지<br/>← 절대 축소 없음]
    TR --> TM[MINIMUM<br/>단어 15m + 리스닝 15m<br/>저녁 35분 스킵 OK<br/>주 2회 한도]

    classDef fullMode fill:#d4edda,stroke:#28a745,color:#000
    classDef reducedMode fill:#fff3cd,stroke:#ffc107,color:#000
    classDef minMode fill:#f8d7da,stroke:#dc3545,color:#000
    class TF fullMode
    class TR reducedMode
    class TM minMode
```

### 3-3. PC 집중 시간 (19:35~21:00)

```mermaid
flowchart TB
    P([PC 집중]) --> PF[FULL<br/>85분 풀 실행<br/>2주 모드 or 일반 모드]
    PF --> PR[REDUCED<br/>30분으로 축소<br/>or 스킵 가능]
    PR --> PM[MINIMUM<br/>스킵]

    classDef fullMode fill:#d4edda,stroke:#28a745,color:#000
    classDef reducedMode fill:#fff3cd,stroke:#ffc107,color:#000
    classDef minMode fill:#f8d7da,stroke:#dc3545,color:#000
    class PF fullMode
    class PR reducedMode
    class PM minMode
```

### 3-4. 저녁 운동

```mermaid
flowchart TB
    E([저녁 운동]) --> EF[FULL<br/>근력 or 유산소 30분<br/>요일별 C5 로테이션]
    EF --> ER[REDUCED<br/>15분<br/>스트레칭+코어만]
    ER --> EM[MINIMUM<br/>스킵 OK<br/>chin tuck은 유지]

    classDef fullMode fill:#d4edda,stroke:#28a745,color:#000
    classDef reducedMode fill:#fff3cd,stroke:#ffc107,color:#000
    classDef minMode fill:#f8d7da,stroke:#dc3545,color:#000
    class EF fullMode
    class ER reducedMode
    class EM minMode
```

### 3-5. OMSCS 기초 학습

```mermaid
flowchart TB
    O([OMSCS]) --> OF[FULL<br/>1시간<br/>수학/딥러닝 신규 진도]
    OF --> OR[REDUCED<br/>30분<br/>복습/영상 시청]
    OR --> OM[MINIMUM<br/>스킵]

    classDef fullMode fill:#d4edda,stroke:#28a745,color:#000
    classDef reducedMode fill:#fff3cd,stroke:#ffc107,color:#000
    classDef minMode fill:#f8d7da,stroke:#dc3545,color:#000
    class OF fullMode
    class OR reducedMode
    class OM minMode
```

### 3-6. 수면

```mermaid
flowchart TB
    S([수면]) --> SF[FULL<br/>23:30 취침<br/>7시간]
    SF --> SR[REDUCED<br/>23:00 취침<br/>7시간 30분<br/>30분 앞당김]
    SR --> SM[MINIMUM<br/>22:00 취침<br/>8시간 30분<br/>회복 우선]

    classDef fullMode fill:#d4edda,stroke:#28a745,color:#000
    classDef reducedMode fill:#fff3cd,stroke:#ffc107,color:#000
    classDef minMode fill:#f8d7da,stroke:#dc3545,color:#000
    class SF fullMode
    class SR reducedMode
    class SM minMode
```

### 3-7. 식단 · chin tuck (모드 무관 고정)

```mermaid
flowchart TB
    Fixed([모드 무관 고정 항목]) --> F1[아침 황제<br/>볶음 덮밥<br/>현미밥+계란+참치<br/>+채소+김<br/>~630kcal 단백40g]
    Fixed --> F2[점심 귀족<br/>회사 샐러드<br/>닭가슴살+채소<br/>+올리브 드레싱<br/>~500kcal 단백30g]
    Fixed --> F3[저녁 빈자<br/>삶은 계란 2~3개<br/>+생채소<br/>19시 이전 종료<br/>~340kcal 단백20g]
    Fixed --> F4[운동 후 WPI<br/>1스쿱+물 250ml<br/>근력일 필수<br/>근합성 골든타임]
    Fixed --> F5[chin tuck 5분<br/>취침 전<br/>경추 케어<br/>매일 강제]
    Fixed --> F6[금지 항목<br/>야식 X<br/>술 X<br/>19시 이후 금식]

    classDef food fill:#fff3cd,stroke:#ffc107,stroke-width:2px,color:#000
    classDef wpi fill:#d1ecf1,stroke:#0c5460,stroke-width:2px,color:#000
    classDef health fill:#cfe2ff,stroke:#0d6efd,stroke-width:2px,color:#000
    classDef forbidden fill:#f8d7da,stroke:#dc3545,stroke-width:2px,color:#000
    class F1,F2,F3 food
    class F4 wpi
    class F5 health
    class F6 forbidden
```

---

## 4. 요일별 로테이션 (TOEFL + PC 집중 시간)

### 4-1. 월요일

```mermaid
flowchart TB
    Mon([월요일]) --> MonAM[아침 30분<br/>리딩<br/>공식 Practice 1세트]
    Mon --> MonPM[저녁 30분<br/>스피킹<br/>Task 1 템플릿 암기<br/>3회 녹음]
    Mon --> MonEx[저녁 운동<br/>근력 상체<br/>+ 유산소 20분]
    Mon --> MonPC[PC 집중 85분<br/>일반: 회사 프로젝트<br/>2주: AI_CONNECT_BRIDGE]

    classDef toeflAM fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef toeflPM fill:#b8daff,stroke:#0d6efd,color:#000
    classDef exercise fill:#d4edda,stroke:#28a745,color:#000
    classDef project fill:#f8d7da,stroke:#dc3545,color:#000
    class MonAM toeflAM
    class MonPM toeflPM
    class MonEx exercise
    class MonPC project
```

### 4-2. 화요일

```mermaid
flowchart TB
    Tue([화요일]) --> TueAM[아침 30분<br/>리스닝<br/>강의 듣기 + 노트테이킹]
    Tue --> TuePM[저녁 30분<br/>라이팅<br/>Academic Discussion 1편<br/>Claude 첨삭]
    Tue --> TueEx[저녁 운동<br/>유산소 40~50분<br/>걷기 or 자전거]
    Tue --> TuePC[PC 집중 85분<br/>일반: GameTrade<br/>2주: AI_CONNECT_BRIDGE]

    classDef toeflAM fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef toeflPM fill:#b8daff,stroke:#0d6efd,color:#000
    classDef exercise fill:#d4edda,stroke:#28a745,color:#000
    classDef project fill:#f8d7da,stroke:#dc3545,color:#000
    class TueAM toeflAM
    class TuePM toeflPM
    class TueEx exercise
    class TuePC project
```

### 4-3. 수요일

```mermaid
flowchart TB
    Wed([수요일]) --> WedAM[아침 30분<br/>리딩<br/>일상 읽기<br/>+ 단어 완성 연습]
    Wed --> WedPM[저녁 30분<br/>스피킹<br/>Task 2,3 통합형<br/>3회 녹음 + 60초 즉흥]
    Wed --> WedEx[저녁 운동<br/>근력 하체<br/>+ 코어]
    Wed --> WedPC[PC 집중 85분<br/>일반: Smart Stock<br/>2주: AI_CONNECT_BRIDGE]

    classDef toeflAM fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef toeflPM fill:#b8daff,stroke:#0d6efd,color:#000
    classDef exercise fill:#d4edda,stroke:#28a745,color:#000
    classDef project fill:#f8d7da,stroke:#dc3545,color:#000
    class WedAM toeflAM
    class WedPM toeflPM
    class WedEx exercise
    class WedPC project
```

### 4-4. 목요일

```mermaid
flowchart TB
    Thu([목요일]) --> ThuAM[아침 30분<br/>리스닝<br/>짧은 대화 · 공지]
    Thu --> ThuPM[저녁 30분<br/>라이팅<br/>통합형 에세이<br/>Claude 첨삭]
    Thu --> ThuEx[저녁 운동<br/>유산소<br/>+ 목·어깨 스트레칭]
    Thu --> ThuPC[PC 집중 85분<br/>일반: ParallelPlay<br/>2주: AI_CONNECT_BRIDGE]

    classDef toeflAM fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef toeflPM fill:#b8daff,stroke:#0d6efd,color:#000
    classDef exercise fill:#d4edda,stroke:#28a745,color:#000
    classDef project fill:#f8d7da,stroke:#dc3545,color:#000
    class ThuAM toeflAM
    class ThuPM toeflPM
    class ThuEx exercise
    class ThuPC project
```

### 4-5. 금요일

```mermaid
flowchart TB
    Fri([금요일]) --> FriAM[아침 30분<br/>리딩<br/>오답 분석<br/>+ 약점 보완]
    Fri --> FriPM[저녁 30분<br/>스피킹<br/>이번 주 녹음 셀프 리뷰<br/>+ 60초 즉흥]
    Fri --> FriEx[저녁 운동<br/>근력 전신]
    Fri --> FriPC[PC 집중 85분<br/>일반: GamePromo<br/>2주: AI_CONNECT_BRIDGE]

    classDef toeflAM fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef toeflPM fill:#b8daff,stroke:#0d6efd,color:#000
    classDef exercise fill:#d4edda,stroke:#28a745,color:#000
    classDef project fill:#f8d7da,stroke:#dc3545,color:#000
    class FriAM toeflAM
    class FriPM toeflPM
    class FriEx exercise
    class FriPC project
```

---

## 5. 주말 스케줄

### 5-1. 토요일 (FULL 모드)

```mermaid
flowchart TB
    S1[7:30~8:00<br/>기상 + 스트레칭 + 아침] --> S2
    S2[8:00~8:30<br/>모의고사 환경 세팅] --> S3
    S3[8:30~11:30<br/>TOEFL 풀 모의고사<br/>TestGlider or ETS<br/>3시간] --> S4
    S4[11:30~12:30<br/>오답 분석 + 약점 정리] --> S5
    S5[12:30~13:30<br/>점심<br/>단백질 + 채소] --> S6
    S6[13:30~15:30<br/>OMSCS 집중<br/>수학/딥러닝 2h] --> S7
    S7[15:30~17:00<br/>2주: AI_CONNECT_BRIDGE 정리<br/>일반: 사이드 프로젝트] --> S8
    S8[17:00~18:30<br/>긴 유산소 60~90분<br/>등산/자전거/조깅] --> S9
    S9([18:30~<br/>저녁 + 자유 시간])

    classDef toefl fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef exercise fill:#d4edda,stroke:#28a745,color:#000
    classDef project fill:#f8d7da,stroke:#dc3545,color:#000
    classDef study fill:#fff3cd,stroke:#ffc107,color:#000
    classDef food fill:#fff3cd,stroke:#ffc107,color:#000

    class S3,S4 toefl
    class S8 exercise
    class S7 project
    class S6 study
    class S5 food
```

> **토요일 REDUCED/MIN 모드일 때**:
> - 모의고사 3h → 2h 단축형 (REDUCED) 또는 단어+리스닝 1h만 (MIN)
> - OMSCS 2h → 1h (REDUCED) 또는 스킵 (MIN)
> - 프로젝트 1.5h → 45분 (REDUCED) 또는 스킵 (MIN)
> - **긴 유산소는 그대로 유지** (회복 효과 큼)

---

### 5-2. 일요일 (FULL 모드)

```mermaid
flowchart TB
    U1[오전<br/>늦잠 OK + 충분한 아침] --> U2
    U2[11:00~12:00<br/>주간 식단 · 운동 · 체중 기록] --> U3
    U3[12:00~13:00<br/>점심] --> U4
    U4[13:00~14:00<br/>TOEFL 단어 복습<br/>+ 다음 주 계획] --> U5
    U5[14:00~15:30<br/>주간 리뷰<br/>Codex 전체 프로젝트 코드 리뷰] --> U6
    U6[15:30~17:00<br/>2주: AI_CONNECT_BRIDGE 정리<br/>일반: Voca/사이드 프로젝트] --> U7
    U7[17:00~18:00<br/>가벼운 산책] --> U8
    U8([18:00~<br/>저녁 + 휴식])

    classDef toefl fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef exercise fill:#d4edda,stroke:#28a745,color:#000
    classDef project fill:#f8d7da,stroke:#dc3545,color:#000
    classDef health fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef food fill:#fff3cd,stroke:#ffc107,color:#000

    class U4 toefl
    class U7 exercise
    class U5,U6 project
    class U2 health
    class U3 food
```

> **일요일 REDUCED/MIN 모드일 때**:
> - 주간 리뷰 30분만
> - 산책 30분
> - 다음 주 일정 점검만 하고 **나머지는 회복 우선**

---

## 6. 4/11 ~ 4/24 · AI_CONNECT_BRIDGE 2주 집중 기간

```mermaid
flowchart TB
    D0([4/11 토 · 집중 시작]) --> D1

    D1[Week 1: 4/11~4/17]
    D1 --> D1Goal[목표<br/>기획서 1개 spec 등록<br/>첫 task 진행<br/>UX 결함 첫 발견]
    D1 --> D1Daily[매일 저녁 PC 시간<br/>19:35~21:00<br/>AI_CONNECT_BRIDGE 실사용]
    D1 --> D1Doc[매일<br/>docs/v0.2.0_feedback.md<br/>개선점 누적]

    D1Goal --> W1End([4/17 금 Week 1 종료])
    D1Daily --> W1End
    D1Doc --> W1End

    W1End --> MidReview[4/18~4/19 주말<br/>중간 정리<br/>1주차 피드백 재분류]

    MidReview --> D2

    D2[Week 2: 4/18~4/24]
    D2 --> D2Goal[목표<br/>추가 개선점 발굴<br/>3왕 핑퐁 실제 검증<br/>MCP 도구 실사용]
    D2 --> D2Daily[매일 저녁 PC 시간<br/>AI_CONNECT_BRIDGE 실사용<br/>회사 업무에도 적용]
    D2 --> D2Doc[매일 피드백 누적<br/>우선순위 태깅<br/>P0/P1/P2]

    D2Goal --> W2End([4/24 금 종료])
    D2Daily --> W2End
    D2Doc --> W2End

    W2End --> Final[4/24 금 저녁<br/>피드백 최종 정리<br/>v0.2.0 백로그 확정]

    Final --> Unlock([4/25 토<br/>사이드 프로젝트 해금<br/>GameTrade 착수 가능])

    Unlock --> V6[v6 마스터 플랜 작성<br/>사용자가 개선점 공유<br/>→ AI가 v6 업데이트]

    subgraph Frozen[기간 중 동결]
        Fz1[GameTrade 코딩 X]
        Fz2[GamePromo 코딩 X]
        Fz3[Smart Stock 코딩 X]
        Fz4[Voca 코딩 X]
        Fz5[ParallelPlay 코딩 X]
        Fz6[InspectNote 코딩 X]
        Fz7[Lite Inspect 코딩 X]
    end

    subgraph Allowed[기간 중 허용]
        Al1[회사 업무 그대로]
        Al2[TOEFL 그대로]
        Al3[건강·운동 그대로]
        Al4[AI_CONNECT_BRIDGE 실사용<br/>+피드백 기록]
        Al5[회사 업무용 코드 분석<br/>dogfooding OK]
    end

    classDef week fill:#fff3cd,stroke:#ffc107,color:#000
    classDef goal fill:#cfe2ff,stroke:#0d6efd,color:#000
    classDef frozen fill:#e2e3e5,stroke:#6c757d,color:#000
    classDef allowed fill:#d4edda,stroke:#28a745,color:#000
    classDef milestone fill:#f8d7da,stroke:#dc3545,color:#000

    class D1,D2 week
    class D1Goal,D1Daily,D1Doc,D2Goal,D2Daily,D2Doc goal
    class Fz1,Fz2,Fz3,Fz4,Fz5,Fz6,Fz7 frozen
    class Al1,Al2,Al3,Al4,Al5 allowed
    class D0,W1End,W2End,Final,Unlock,V6 milestone
```

---

## 읽는 법

1. **첫 판단**: 섹션 0 전체 조감도로 오늘 모드 빠르게 정함
2. **컨디션 애매하면**: 섹션 1 플로우로 정확히 판정
3. **평일 시간표 확인**: 섹션 2 (FULL) → 섹션 3 (REDUCED/MIN 축소)
4. **오늘 뭘 할지 (요일별)**: 섹션 4
5. **주말**: 섹션 5
6. **지금 2주 모드 중이면**: 섹션 6 참조

색상 코드:
- 🟦 파란색 = TOEFL
- 🟩 초록색 = 운동·허용
- 🟨 노란색 = 식단·REDUCED 모드
- 🟥 빨간색 = 프로젝트·MINIMUM 모드·마일스톤
- ⬜ 회색 = 회사 업무·동결·회복
