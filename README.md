# NeuroStyleSeq

## 🎼 DualMemoryMusic

**신경과학 영감을 반영한 작곡가 스타일 전이**  
**Decoupled Temporal-Ordinal Modeling 기반 클래식 피아노 스타일 전이 프레임워크**

---

## 1. 동기 (Motivation)

클래식 피아노 음악은 다음 특성을 가집니다.

- 장거리 구조적 일관성 (Long-range structural coherence)
- 계층적 프레이즈 조직 (Hierarchical phrase organization)
- 작곡가 고유의 화성/텍스처 시그니처
- 템포가 변해도 유지되는 구조적 정체성

기존 시퀀스 모델은 보통 아래 정보를 하나의 잠재공간에 함께 얽어 학습합니다.

- 이벤트 정체성 (`what`)
- 시간 간격 (`when`)
- 스타일 맥락 (`who`)

하지만 신경과학 관점에서는 다음 분리가 제안됩니다.

- 순서 정보(order)와 시간 간격(interval) 정보의 분리 인코딩
- 시퀀스 정체성을 유지하는 별도 목표/맥락 표현
- 속도 변화에도 안정적인 회상을 가능하게 하는 시간 페이싱 메커니즘

### 핵심 가설 (Core Hypothesis)

클래식 작곡가 스타일의 일관성은 아래 세 요소를 **명시적으로 분리(decoupling)** 할 때 향상됩니다.

- **Ordinal Memory**: 이벤트 진행 순서 기억
- **Temporal Interval Memory**: 이벤트 간 시간 간격 기억
- **Persistent Stylistic Context**: 지속되는 스타일 맥락

이 프로젝트는 이 원리를 구조화된 스타일 전이 프레임워크로 구현합니다.

---

## 2. 문제 정의 (Problem Definition)

작곡가 스타일 전이를 다단계 제약 변환 문제로 정의합니다.

- 입력: 원본 곡 `X`
- 조건: 목표 작곡가 `c_t`
- 출력: 스타일 전이 결과 `X^c_t`

목표 조건:

- 곡의 핵심 콘텐츠(content core)는 보존
- 화성, 텍스처, 장식 통계는 목표 작곡가 `c_t`의 분포와 정렬

---

## 3. 단계 기반 스타일 전이 (Stage-Based Style Transfer)

스타일을 점진적으로 침습적인 변환으로 분해합니다.

### 🔹 Stage 1 - Re-Harmonization

**고정(Fixed)**

- 멜로디 음고 윤곽
- 온셋 타이밍
- 프레이즈 분할

**변경(Modified)**

- 화성 진행
- 코드 보이싱
- 코드 밀도

**목적함수**

```text
min L_melody + L_rhythm + λ L_style(harmony)
```

이 단계는 "콘텐츠 보존"의 기준을 가장 명확하게 정의합니다.

### 🔹 Stage 2 - Re-Texturization

**고정(Fixed)**

- 멜로디 핵심
- 코드 루트

**변경(Modified)**

- 왼손 패턴
- 성부 밀도
- 음역 분산
- 반주 아키타입

**주요 스타일 지표 예시**

- 아르페지오 출현 확률
- 블록 코드 사용 가능도
- 옥타브 더블링 빈도
- 시간축 텍스처 분산

### 🔹 Stage 3 - Ornamentation

아래 요소를 제어된 방식으로 삽입합니다.

- 그레이스톤 유사 음형
- 패싱 장식음
- 빠른 패시지 조각
- 도약 분포 조정

단, 구조적 핵심음(structural tones)은 보존합니다.

---

## 4. 모델 아키텍처 (Model Architecture)

### 🧠 상위 원리

본 프로젝트는 **Dual Memory Architecture**를 사용합니다.

- Ordinal Memory Stream
- Temporal Interval Stream
- Persistent Style State

### 4.1 Dual-Stream Encoding

기존처럼 토큰을 단일 형태로 인코딩하는 대신,

```text
x_t = (pitch, duration)
```

아래처럼 분리합니다.

- 이벤트 토큰 `e_t`
- 간격 토큰 `Δt_t`

간격 토큰은 시간 기저 함수(time-basis function)로 인코딩합니다.

```text
ϕ(Δt) = { exp(-(Δt - μ_k)^2 / σ_k^2) }
```

동기: time-cell tuning 특성에서 영감.

### 4.2 Persistent Style State

작곡가별 전역 잠재벡터 `z_c`를 도입합니다.

```text
z_c(t) = α z_c(t-1) + (1-α) f(h_t)
```

- `h_t`: hidden state
- `α`: 지속성(persistence) 제어 계수

이는 목표/맥락 기억 유지(goal/context maintenance)를 모사합니다.

### 4.3 Hebbian-Inspired Regularization

상관 기반 정규화를 추가합니다.

```text
L_hebb = Σ_t sim(h_t, h_{t+1})
```

인접 시점 상태가 안정적인 assembly를 형성하도록 유도하며,
스파이킹 동역학 없이 가소성 원리를 근사합니다.

---

## 5. 학습 목표 (Training Objectives)

```text
L = L_reconstruction + λ1 L_style + λ2 L_hebb
```

- `L_reconstruction`: 콘텐츠 보존
- `L_style`: 목표 작곡가 분포 정렬
- `L_hebb`: 시간적 assembly 안정화

---

## 6. 실험 설계 (Experimental Design)

### 🎯 Baseline Models

- Vanilla Transformer (composer token condition)
- Conditional diffusion (optional)

### 🔍 Ablation Study

| Variant | Interval Stream | Persistent Context | Hebbian Term |
|---|---|---|---|
| Full | ✓ | ✓ | ✓ |
| -Interval | ✗ | ✓ | ✓ |
| -Context | ✓ | ✗ | ✓ |
| -Hebb | ✓ | ✓ | ✗ |

---

## 7. 평가 지표 (Evaluation Metrics)

### 🎵 Content Preservation

- Pitch contour similarity
- IOI similarity
- Phrase boundary agreement

### 🎼 Style Adoption

- Composer classifier accuracy
- Harmonic entropy distance
- Texture statistics KL divergence

### 🧱 Structural Coherence

- Motif recurrence index
- Long-range tonal drift
- Pitch range variance alignment

### ⏱ Tempo Robustness

입력 템포를 `×0.7`, `×1.3`으로 스케일링한 뒤 아래를 평가합니다.

- 스타일 유지 안정성
- 콘텐츠 유사도 안정성

이 실험은 tempo-resilient memory 메커니즘에서 영감을 받습니다.

---

## 8. 기대 기여 (Expected Contributions)

- 다단계 클래식 작곡가 스타일 전이의 형식적 정의
- 시간-순서 분리 모델링의 실증 검증
- 지속적 스타일 맥락 모델링 제안
- 템포 강건 구조 전이 평가 프로토콜 제시

---

## 9. 연구 철학 (Research Philosophy)

이 프로젝트는 생물학적 뉴런을 직접 시뮬레이션하지 않습니다.

대신,

- 신경 기억 이론에서 기능적 원리를 추출하고
- 이를 안정적인 딥러닝 귀납 편향(inductive bias)으로 번역하며
- 구조화된 음악 도메인에서 그 필요성을 검증합니다.

---

## 10. 프로젝트 로드맵 (Project Roadmap)

- [ ] 데이터셋 전처리
- [ ] Dual-stream encoder 구현
- [ ] Stage 1 학습
- [ ] Stage 2 확장
- [ ] Ablation study
- [ ] 템포 강건성 실험
- [ ] 논문 초안 제출
