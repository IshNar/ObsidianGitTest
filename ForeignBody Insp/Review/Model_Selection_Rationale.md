# 인공지능 모델 선정 타당성 보고서 (Model Selection Rationale)

본 문서는 바이알 이물 검사 시스템의 분류(Classification) 모델로 **EfficientNet-B0**를 선정하게 된 기술적 배경과 기존 모델(MobileNet, ResNet) 대비 우수성, 그리고 최신 모델들(ViT, ConvNeXt 등)과의 비교 분석을 바탕으로 한 타당성을 기술합니다.

---

## 1. 모델 아키텍처 진화 과정 (Evolution Path)

시스템의 요구사항 변화에 따른 모델 채택 역사는 다음과 같습니다.

### 1차: MobileNet (속도 우선)
*   **특징**: Depthwise Separable Convolution을 사용하여 파라미터 수를 극단적으로 줄임.
*   **한계**: 연산량은 적으나, 미세한 이물(Dust vs. Particle)의 미묘한 텍스처 특징을 잡아내는 정확도가 부족함.

### 2차: ResNet50 (신뢰성 우선)
*   **특징**: Skip Connection을 통해 깊은 망 학습 가능. 산업계의 표준적인 모델.
*   **한계**: 파라미터 수(~25M)와 연산량(~4.1B FLOPS)이 많아 실시간 검사 시 Tact Time 확보에 불리하며, 모바일/엣지 환경에서 무거움.

### 3차: EfficientNet-B0 (최적의 균형 - 현행)
*   **특징**: **Compound Scaling** 기법을 도입하여 모델의 깊이(Depth), 너비(Width), 해상도(Resolution)를 최적의 비율로 동시 확장.
*   **장점**: ResNet50보다 약 5배 적은 파라미터로 더 높은 정확도를 달성.

---

## 2. EfficientNet-B0 핵심 기술 상세 (Deep Dive)

### 2.1 복합 스케일링 (Compound Scaling)
기존 모델들은 모델의 성능을 높이기 위해 다음 중 **하나의 요소만** 확장하는 것이 일반적이었습니다.
*   **Depth ($d$)**: 계층을 더 깊게 쌓음 (예: ResNet-50 → ResNet-101).
*   **Width ($w$)**: 채널 수를 늘림.
*   **Resolution ($r$)**: 입력 이미지 해상도를 높임.

**EfficientNet**은 이 세 가지 요소가 서로 유기적으로 연결되어 있다는 점에 착안하여, 하나의 복합 계수($\phi$)를 통해 **동시에 확장**하는 공식을 제안했습니다.
$$depth: d = \alpha^\phi, \quad width: w = \beta^\phi, \quad resolution: r = \gamma^\phi$$
(단, $\alpha \cdot \beta^2 \cdot \gamma^2 \approx 2$, $\alpha, \beta, \gamma \ge 1$)

이 기법을 통해 EfficientNet-B0는 매우 적은 연산량(0.39B FLOPS)으로도 고해상도 이미지의 세밀한 특징을 효율적으로 포착할 수 있습니다.

### 2.2 MBConv (Mobile Inverted Bottleneck Convolution)
EfficientNet의 기본 빌딩 블록은 MobileNetV2에서 영감을 얻은 **MBConv** 구조입니다.
1.  **Inverted Residual**: 채널을 먼저 확장한 뒤 연산하고 다시 줄이는 구조로, 정보 손실을 최소화하면서 연산 효율을 극대화합니다.
2.  **Squeeze-and-Excitation (SE) Optimization**: 채널 간의 상호작용을 계산하여 중요한 특징 채널에 가중치를 부여하는 **주의 집중(Attention)** 메커니즘이 기본 포함되어 있습니다.
3.  **Swish Activation**: ReLU보다 매끄러운(Smooth) 특성을 가진 Swish 함수를 사용하여 심층 신경망 학습 시 기울기 소실 문제를 완화하고 성능을 개선했습니다.

---

## 3. 기술적 지표 비교 (Technical Metrics)

| 모델명 | Parameters | FLOPS | Top-1 Accuracy (ImageNet) | 비고 |
| :--- | :--- | :--- | :--- | :--- |
| MobileNet-V2 | 3.4M | 0.3B | 72.0% | 가장 빠르나 정확도 낮음 |
| ResNet-50 | 25.6M | 4.1B | 76.0% | 무겁고 느림 |
| **EfficientNet-B0** | **5.3M** | **0.39B** | **77.1%** | **최적의 가성비 (현 모델)** |
| **ConvNeXt-Tiny** | 28M | 4.5B | 82.1% | 성능은 좋으나 무거움 (**[컨브넥스트 타이니]**) |
| **ViT-Base** | 86M | 17B | 77.9% | 데이터가 충분해야 함 (**[빗 베이스 / 브이아이티 베이스]**) |

---

## 3. 최신 아키텍처와의 비교 및 미채택 사유

현시점의 최신 모델인 InternViT, CoCa, ConvNeXt 등과 비교 시 EfficientNet-B0를 유지하는 핵심 사유는 다음과 같습니다.

### 3.1 실시간성 및 하드웨어 제약 (Latency)
*   **InternViT / CoCa**: 수십억 개(Billion 단위)의 파라미터를 갖는 파운데이션 모델로, 고사양 서버급 GPU가 필요합니다. 바이알 검사의 실시간 Tact Time(평균 100ms 이내)을 맞추기에는 연산량이 너무 많습니다.
*   **Swin Transformer / ViT**: Transformer 기반 모델은 전역적인 특징을 잘 잡지만, 이물(ROI) 분류와 같은 국부적인 텍스처 인식에는 CNN(EfficientNet)의 **귀납적 편향(Inductive Bias)**이 적은 데이터셋 환경에서 더 효과적입니다.

### 3.2 데이터 효율성
*   **ViT 계열**: 충분한 성능을 내기 위해 대규모 데이터가 필요합니다. 산업 현장에서 이물 이미지는 상대적으로 수집이 어렵고(Imbalanced Data), 증강(Augmentation)에 의존해야 하므로 EfficientNet의 효율성이 더 높습니다.

### 3.3 최적화 및 배포 편의성
*   EfficientNet-B0는 **ONNX, OpenVINO, TensorRT** 등 상용 가속 엔진에서 가장 잘 지원되는 검증된 모델입니다. 특히 CPU 환경에서도 딥러닝 추론을 원활하게 수행할 수 있는 수준의 가벼움을 제공합니다.

---

---

## 4. 부록: 모델 명칭 및 용어 정리 (Terminology)

| 모델명 | 풀네임 및 유래 | 발음 가이드 |
| :--- | :--- | :--- |
| **EfficientNet** | **Efficient** + **Network**. 연산 효율성을 극대화한 모델. | 에피션트넷 |
| **ConvNeXt** | **Conv**olution + **NeXt** generation. 차세대 컨볼루션 망. | 컨브넥스트 |
| **ViT** | **Vi**sion **T**ransformer. 구글에서 제안한 영상 처리용 트랜스포머. | 빗 / 브이아이티 |
| **MBConv** | **M**obile Inverted **B**ottleneck **Conv**olution. 효율적 연산 블록. | 엠비 컨브 |

---

## 5. 결론 (Conclusion)

EfficientNet-B0는 **"최소한의 자원으로 최대한의 정확도"**를 내야 하는 실시간 이물 검사 시스템에 가장 타당한 선택입니다. 

1.  **ResNet50 대비**: 정확도는 높이면서 연산량은 1/10 수준으로 절감.
2.  **MobileNet 대비**: 연산량 차이는 미미하나, 고수준 특징 추출 능력으로 분류 정확도 대폭 향상.
3.  **최신 ViT 대비**: 적은 학습 데이터로도 안정적인 수렴이 가능하며 국소적 특징(이물) 포착에 유리함.

> [!NOTE]
> 향후 하드웨어 업그레이드나 대규모 데이터 확보 시, EfficientNet-V2 또는 실시간성에 최적화된 EfficientFormer 등의 도입을 검토할 예정입니다.
