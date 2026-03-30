# 인공지능 모델 선정 타당성 보고서 (Model Selection Rationale)

본 문서는 바이알 이물 검사 시스템의 분류(Classification) 모델로 **EfficientNet-B0**를 기본 선정하고, 추가로 **RepViT-M0.9**와 **MobileOne-S1**을 대안으로 제공하게 된 기술적 배경과 비교 분석을 기술합니다.

---

## 1. 모델 아키텍처 진화 과정 (Evolution Path)

시스템의 요구사항 변화에 따른 모델 채택 역사는 다음과 같습니다.

### 1차: MobileNet (속도 우선)
- **특징**: Depthwise Separable Convolution으로 파라미터 수를 극단적으로 줄임.
- **한계**: 연산량은 적으나, 미세한 이물(Dust vs. Particle)의 미묘한 텍스처 특징을 잡아내는 정확도가 부족.

### 2차: ResNet50 (신뢰성 우선)
- **특징**: Skip Connection을 통해 깊은 망 학습 가능. 산업계 표준 모델.
- **한계**: 파라미터 수(~25M)와 연산량(~4.1B FLOPS)이 많아 실시간 Tact Time 확보에 불리.

### 3차: EfficientNet-B0 (최적 균형 — 현행 기본)
- **특징**: Compound Scaling으로 깊이/너비/해상도를 동시 최적화.
- **장점**: ResNet50보다 약 5배 적은 파라미터로 더 높은 정확도 달성.

### 4차: RepViT-M0.9 / MobileOne-S1 (추가 대안)
- 최신 경량 아키텍처로, EfficientNet-B0보다 추론 속도가 2~3배 빠르면서 정확도도 유사하거나 더 높음.
- `timm` 라이브러리 의존성이 있어 선택적으로 제공.

---

## 2. 현재 시스템에서 선택 가능한 모델 비교

| 모델명 | Parameters | FLOPS | ImageNet Top-1 | 추론 속도 (상대) | 비고 |
|--------|-----------|-------|----------------|-----------------|------|
| **EfficientNet-B0** | 5.3M | 0.39B | 77.1% | 1x (기준) | **기본 모델**. ONNX/OpenVINO 지원 최적. |
| **RepViT-M0.9** | ~5M | ~0.9B | ~79% | ~2x 빠름 | **추천 대안**. ViT 기반이지만 경량화된 구조. timm 필요. |
| **MobileOne-S1** | ~4M | ~0.3B | ~76% | ~3x 빠름 | **최고속**. 재파라미터화(Reparameterization)로 추론 시 단순 구조. timm 필요. |

### 2.1 모델별 장단점 요약

#### EfficientNet-B0 (기본)
- **장점**: ONNX, OpenVINO, TensorRT 등 모든 상용 가속 엔진에서 검증됨. CPU 환경에서도 원활한 추론 가능.
- **단점**: RepViT 대비 추론 속도가 느림.
- **적합 상황**: 안정성을 최우선시하거나, OpenVINO NPU를 사용할 때.

#### RepViT-M0.9 (추천)
- **장점**: Vision Transformer의 전역 특징 추출 능력 + CNN의 로컬 특징 추출 능력을 결합. B0보다 정확도 높고 속도도 빠름.
- **단점**: timm 라이브러리 의존. ONNX 변환 시 일부 연산자 호환성 확인 필요.
- **적합 상황**: GPU 사용 가능하고, 정확도와 속도를 모두 원할 때.

#### MobileOne-S1 (최고속)
- **장점**: 추론 시 재파라미터화로 단순 Conv 스택이 되어 극도로 빠름.
- **단점**: B0보다 정확도가 약간 낮을 수 있음. timm 의존.
- **적합 상황**: Tact Time이 매우 짧아야 하는 고속 라인.

---

## 3. EfficientNet-B0 핵심 기술 상세

### 3.1 복합 스케일링 (Compound Scaling)

기존 모델들은 성능을 높이기 위해 깊이(Depth), 너비(Width), 해상도(Resolution) 중 **하나만** 확장하는 것이 일반적이었다.

EfficientNet은 이 세 요소가 서로 유기적으로 연결되어 있다는 점에 착안하여, 하나의 복합 계수(φ)를 통해 **동시에 확장**하는 공식을 제안했다:

```
depth: d = α^φ
width: w = β^φ
resolution: r = γ^φ
(단, α × β² × γ² ≈ 2, α, β, γ ≥ 1)
```

이를 통해 매우 적은 연산량(0.39B FLOPS)으로도 고해상도 이미지의 세밀한 특징을 효율적으로 포착한다.

### 3.2 MBConv (Mobile Inverted Bottleneck Convolution)

EfficientNet의 기본 빌딩 블록:
1. **Inverted Residual**: 채널을 먼저 확장한 뒤 연산하고 다시 줄이는 구조로, 정보 손실을 최소화하면서 연산 효율을 극대화.
2. **Squeeze-and-Excitation (SE)**: 채널 간 상호작용을 계산하여 중요한 특징 채널에 가중치를 부여하는 주의 집중(Attention) 메커니즘.
3. **Swish Activation**: ReLU보다 매끄러운 특성으로 심층 신경망 학습 시 기울기 소실 문제를 완화.

---

## 4. 기존 및 최신 모델 대비 비교

| 모델명 | Parameters | FLOPS | Top-1 (ImageNet) | 비고 |
|--------|-----------|-------|-------------------|------|
| MobileNet-V2 | 3.4M | 0.3B | 72.0% | 가장 빠르나 정확도 낮음 |
| ResNet-50 | 25.6M | 4.1B | 76.0% | 무겁고 느림 |
| **EfficientNet-B0** | **5.3M** | **0.39B** | **77.1%** | **현행 기본 모델** |
| ConvNeXt-Tiny | 28M | 4.5B | 82.1% | 정확하지만 무거움 |
| ViT-Base | 86M | 17B | 77.9% | 대규모 데이터 필요 |
| Swin-Tiny | 28M | 4.5B | 81.2% | Transformer 계열, 무거움 |

### 4.1 최신 대형 모델 미채택 사유

#### 실시간성 제약 (Latency)
- **InternViT / CoCa**: 수십억 파라미터의 파운데이션 모델로, 서버급 GPU가 필요. 실시간 Tact Time(평균 100ms 이내)을 맞출 수 없음.
- **Swin Transformer / ViT**: Transformer 기반은 전역 특징 포착에 강하지만, 이물(ROI) 분류 같은 **국부적 텍스처 인식**에는 CNN의 귀납적 편향(Inductive Bias)이 적은 데이터에서 더 효과적.

#### 데이터 효율성
- ViT 계열은 충분한 성능을 내기 위해 대규모 데이터가 필요. 산업 현장에서 이물 이미지는 수집이 어렵고(불균형 데이터), 증강에 의존해야 하므로 EfficientNet의 효율성이 더 높음.

#### 배포 편의성
- EfficientNet-B0는 ONNX, OpenVINO, TensorRT에서 가장 잘 지원되는 검증된 모델. CPU 환경에서도 원활한 추론 가능.

---

## 5. 부록: 모델 명칭 및 용어 정리

| 모델명 | 풀네임 및 유래 | 발음 가이드 |
|--------|---------------|------------|
| **EfficientNet** | Efficient + Network. 연산 효율성 극대화 | 에피션트넷 |
| **RepViT** | Reparameterized Vision Transformer. 재파라미터화 경량 ViT | 렙빗 |
| **MobileOne** | Apple에서 제안한 모바일 최적화 모델 | 모바일원 |
| **ConvNeXt** | Convolution + Next generation. 차세대 CNN | 컨브넥스트 |
| **ViT** | Vision Transformer. 구글 제안 영상 처리 트랜스포머 | 빗 / 브이아이티 |
| **MBConv** | Mobile Inverted Bottleneck Convolution | 엠비 컨브 |
| **timm** | PyTorch Image Models. Hugging Face에서 관리하는 모델 라이브러리 | 팀 |

---

## 6. 결론 (Conclusion)

EfficientNet-B0는 **"최소한의 자원으로 최대한의 정확도"**를 내야 하는 실시간 이물 검사 시스템에 가장 안정적인 선택이다.

1. **ResNet50 대비**: 정확도는 높이면서 연산량은 1/10 수준으로 절감.
2. **MobileNet 대비**: 연산량 차이는 미미하나, 고수준 특징 추출 능력으로 분류 정확도 대폭 향상.
3. **최신 ViT 대비**: 적은 학습 데이터로도 안정적인 수렴이 가능하며 국소적 특징(이물) 포착에 유리.

더 높은 속도가 필요한 경우 **RepViT-M0.9** (정확도+속도 균형) 또는 **MobileOne-S1** (최고속)을 선택할 수 있으며, UI의 학습 탭에서 모델 아키텍처를 변경하면 된다.

> **향후 계획**: 하드웨어 업그레이드나 대규모 데이터 확보 시, EfficientNet-V2 또는 EfficientFormer 등의 도입을 검토할 예정.
