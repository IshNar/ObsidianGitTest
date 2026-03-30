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

> **cf. 용어 해설**
> - **Depthwise Separable Convolution**: 표준 합성곱을 깊이별(Depthwise) 합성곱과 점별(Pointwise) 합성곱으로 분리하여 연산량과 파라미터 수를 대폭 줄이는 합성곱 기법.
> - **파라미터(Parameter)**: 신경망이 학습 과정에서 조정하는 가중치(Weight)와 편향(Bias) 등의 수치. 파라미터 수가 많을수록 모델이 크고 메모리를 많이 차지함.
> - **Skip Connection**: 신경망의 특정 층을 건너뛰어 입력을 출력에 직접 더하는 연결 구조. 깊은 망에서 기울기 소실을 방지하고 학습 안정성을 높임(ResNet에서 도입).
> - **Compound Scaling**: EfficientNet에서 제안된 스케일링 기법으로, 네트워크의 깊이·너비·입력 해상도를 하나의 복합 계수로 동시에 확장하여 효율적으로 성능을 높이는 방법.
> - **깊이(Depth)**: 신경망에서 쌓인 층(Layer)의 수. 깊이가 깊을수록 복잡한 특징을 학습할 수 있으나 연산량과 학습 난이도가 증가함.
> - **너비(Width)**: 각 층에서 사용되는 필터(채널)의 수. 너비가 넓을수록 더 다양한 특징을 동시에 포착할 수 있음.
> - **해상도(Resolution)**: 신경망에 입력되는 이미지의 가로×세로 픽셀 크기. 해상도가 높을수록 세밀한 특징 포착이 가능하나 연산량이 급증함.
> - **FLOPS (Floating Point Operations Per Second)**: 부동소수점 연산 횟수. 모델의 연산량(계산 복잡도)을 나타내는 지표로, 값이 낮을수록 더 가벼운 모델임.
> - **Tact Time**: 제조 라인에서 하나의 제품을 처리하는 데 허용되는 시간 주기. 검사 시스템에서는 1개 바이알의 촬영~판정까지 소요되는 시간을 의미.
> - **timm**: PyTorch Image Models의 약자. Hugging Face에서 관리하는 오픈소스 라이브러리로, 수백 가지 사전 학습된 이미지 분류 모델을 제공.
> - **재파라미터화(Reparameterization)**: 학습 시 사용한 복잡한 다중 분기 구조를 추론 시 단일 합성곱으로 수학적으로 합쳐 추론 속도를 극대화하는 기법.
> - **ViT (Vision Transformer)**: 자연어 처리의 Transformer 구조를 이미지 인식에 적용한 모델. 이미지를 패치(Patch) 단위로 나누어 전역적 특징을 학습함.
> - **CNN (Convolutional Neural Network)**: 합성곱 신경망. 합성곱 필터를 사용하여 이미지의 국부적(Local) 패턴을 계층적으로 학습하는 신경망 구조.
> - **경량화(Lightweight)**: 모델의 파라미터 수와 연산량을 줄여 제한된 하드웨어(CPU, 모바일, 엣지 디바이스)에서도 빠르게 동작할 수 있도록 최적화하는 것.
> - **Backbone**: 신경망에서 입력 이미지로부터 특징(Feature)을 추출하는 주요 몸체(본체) 네트워크. 분류 헤드나 디텍션 헤드 앞단에 위치함.

---

## 2. 현재 시스템에서 선택 가능한 모델 비교

| 모델명 | Parameters | FLOPS | ImageNet Top-1 | 추론 속도 (상대) | 비고 |
|--------|-----------|-------|----------------|-----------------|------|
| **EfficientNet-B0** | 5.3M | 0.39B | 77.1% | 1x (기준) | **기본 모델**. ONNX/OpenVINO 지원 최적. |
| **RepViT-M0.9** | ~5M | ~0.9B | ~79% | ~2x 빠름 | **추천 대안**. ViT 기반이지만 경량화된 구조. timm 필요. |
| **MobileOne-S1** | ~4M | ~0.3B | ~76% | ~3x 빠름 | **최고속**. 재파라미터화(Reparameterization)로 추론 시 단순 구조. timm 필요. |

> **cf. 용어 해설**
> - **Parameters**: 모델이 학습을 통해 조정하는 가중치의 총 개수. M은 백만(Million) 단위를 의미하며, 모델의 크기와 메모리 사용량을 나타냄.
> - **FLOPS**: Floating Point Operations Per Second. 모델이 한 번의 추론에 수행하는 부동소수점 연산 횟수. B는 10억(Billion) 단위.
> - **ImageNet**: 1,000개 클래스, 약 120만 장의 이미지로 구성된 대규모 이미지 분류 벤치마크 데이터셋. 모델 성능 비교의 표준으로 사용됨.
> - **Top-1 Accuracy**: 모델이 예측한 가장 높은 확률의 클래스가 정답과 일치하는 비율. ImageNet에서의 분류 정확도를 나타내는 대표 지표.
> - **ONNX**: Open Neural Network Exchange. 다양한 딥러닝 프레임워크 간 모델 호환을 위한 개방형 표준 포맷.
> - **OpenVINO**: Intel에서 개발한 추론 최적화 툴킷. Intel CPU/GPU/NPU에서 딥러닝 모델의 추론 속도를 극대화함.
> - **TensorRT**: NVIDIA에서 개발한 고성능 딥러닝 추론 최적화 엔진. NVIDIA GPU에서 최적화된 추론을 제공.
> - **NPU**: Neural Processing Unit. 딥러닝 연산에 특화된 하드웨어 가속 프로세서.
> - **CPU**: Central Processing Unit. 범용 중앙 처리 장치. GPU 없이도 추론이 가능하나 속도가 느림.
> - **GPU**: Graphics Processing Unit. 병렬 연산에 특화된 그래픽 처리 장치로, 딥러닝 학습 및 추론을 가속함.

### 2.1 모델별 장단점 요약

#### EfficientNet-B0 (기본)
- **장점**: ONNX, OpenVINO, TensorRT 등 모든 상용 가속 엔진에서 검증됨. CPU 환경에서도 원활한 추론 가능.
- **단점**: RepViT 대비 추론 속도가 느림.
- **적합 상황**: 안정성을 최우선시하거나, OpenVINO NPU를 사용할 때.

> **cf. 용어 해설**
> - **ONNX**: Open Neural Network Exchange. 프레임워크 간 모델 이식을 위한 개방형 중간 표현 포맷.
> - **OpenVINO**: Intel의 딥러닝 추론 최적화 툴킷. CPU/GPU/VPU/NPU 등 Intel 하드웨어에서 추론을 가속.
> - **TensorRT**: NVIDIA GPU 전용 딥러닝 추론 최적화 라이브러리. 그래프 최적화, 양자화 등을 통해 추론 속도를 극대화.
> - **CPU**: Central Processing Unit. 범용 프로세서로, 별도 가속기 없이도 모델 추론이 가능한 환경.
> - **가속 엔진(Acceleration Engine)**: 딥러닝 모델의 추론 속도를 높이기 위해 하드웨어에 맞게 연산을 최적화해주는 소프트웨어 도구(OpenVINO, TensorRT 등).
> - **추론(Inference)**: 학습이 완료된 모델에 새로운 입력 데이터를 넣어 예측 결과를 얻는 과정. 학습(Training)의 반대 개념.

#### RepViT-M0.9 (추천)
- **장점**: Vision Transformer의 전역 특징 추출 능력 + CNN의 로컬 특징 추출 능력을 결합. B0보다 정확도 높고 속도도 빠름.
- **단점**: timm 라이브러리 의존. ONNX 변환 시 일부 연산자 호환성 확인 필요.
- **적합 상황**: GPU 사용 가능하고, 정확도와 속도를 모두 원할 때.

> **cf. 용어 해설**
> - **Vision Transformer**: 이미지를 고정 크기 패치로 분할한 뒤 Transformer의 Self-Attention 메커니즘으로 전역적 관계를 학습하는 모델 구조.
> - **CNN (Convolutional Neural Network)**: 합성곱 필터를 통해 이미지의 국부적 패턴(에지, 텍스처 등)을 계층적으로 추출하는 신경망.
> - **전역 특징(Global Feature)**: 이미지 전체에 걸친 맥락적·장거리(Long-range) 관계 정보. Transformer 계열이 잘 포착함.
> - **로컬 특징(Local Feature)**: 이미지의 작은 영역(커널 크기) 내에서 추출되는 국부적 패턴(에지, 코너, 텍스처 등). CNN이 효과적으로 학습함.
> - **timm**: PyTorch Image Models. 수백 가지 사전 학습 모델과 유틸리티를 제공하는 오픈소스 라이브러리.
> - **ONNX**: Open Neural Network Exchange. 모델을 프레임워크 독립적인 중간 표현으로 변환하여 다양한 런타임에서 실행할 수 있게 하는 표준 포맷.
> - **연산자(Operator)**: 신경망 그래프에서 수행되는 개별 연산 단위(Conv, MatMul, Softmax 등). ONNX 변환 시 대상 런타임이 해당 연산자를 지원하는지 호환성 확인이 필요.

#### MobileOne-S1 (최고속)
- **장점**: 추론 시 재파라미터화로 단순 Conv 스택이 되어 극도로 빠름.
- **단점**: B0보다 정확도가 약간 낮을 수 있음. timm 의존.
- **적합 상황**: Tact Time이 매우 짧아야 하는 고속 라인.

> **cf. 용어 해설**
> - **재파라미터화(Reparameterization)**: 학습 시 병렬 분기(다중 브랜치) 구조를 추론 시 하나의 합성곱 연산으로 수학적으로 병합하여 추론 지연을 최소화하는 기법.
> - **Conv(Convolution)**: 합성곱 연산. 입력 데이터에 커널(필터)을 슬라이딩하며 특징 맵을 생성하는 신경망의 기본 연산.
> - **스택(Stack)**: 동일하거나 유사한 구조의 층(Layer)을 여러 개 순차적으로 쌓아 올린 구성. MobileOne에서는 재파라미터화 후 단순 Conv 층들의 직렬 스택이 됨.
> - **timm**: PyTorch Image Models. MobileOne-S1 모델을 불러오기 위해 필요한 사전 학습 모델 라이브러리.
> - **Tact Time**: 생산 라인에서 단위 제품 하나를 처리하는 데 할당되는 시간. 고속 라인에서는 Tact Time이 극도로 짧아 초고속 추론이 필수.

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

> **cf. 용어 해설**
> - **복합 스케일링(Compound Scaling)**: 네트워크의 깊이·너비·해상도를 단일 복합 계수(φ)로 동시에 제어하여, 자원 대비 성능을 최대화하는 스케일링 전략.
> - **φ(Phi)**: 복합 스케일링에서 사용되는 사용자 지정 복합 계수. φ 값을 키우면 깊이·너비·해상도가 동시에 증가하여 더 큰(더 정확한) 모델이 됨.
> - **α(Alpha)**: 복합 스케일링 공식에서 깊이(Depth) 스케일링을 제어하는 기본 계수. 그리드 서치로 결정됨.
> - **β(Beta)**: 복합 스케일링 공식에서 너비(Width) 스케일링을 제어하는 기본 계수.
> - **γ(Gamma)**: 복합 스케일링 공식에서 해상도(Resolution) 스케일링을 제어하는 기본 계수.
> - **Depth (깊이)**: 신경망의 층 수. 깊이를 늘리면 더 복잡한 고수준 특징을 학습할 수 있으나 학습 난이도와 연산량이 증가.
> - **Width (너비)**: 각 층의 채널(필터) 수. 너비를 늘리면 더 다양한 세밀한 특징을 포착하지만 메모리와 연산량이 증가.
> - **Resolution (해상도)**: 입력 이미지의 픽셀 크기. 해상도를 높이면 세밀한 패턴을 인식할 수 있으나 연산량이 제곱으로 증가.
> - **FLOPS**: Floating Point Operations. 모델의 한 번 추론에 필요한 부동소수점 연산 횟수로, 모델의 계산 비용을 나타내는 지표.

### 3.2 MBConv (Mobile Inverted Bottleneck Convolution)

EfficientNet의 기본 빌딩 블록:
1. **Inverted Residual**: 채널을 먼저 확장한 뒤 연산하고 다시 줄이는 구조로, 정보 손실을 최소화하면서 연산 효율을 극대화.
2. **Squeeze-and-Excitation (SE)**: 채널 간 상호작용을 계산하여 중요한 특징 채널에 가중치를 부여하는 주의 집중(Attention) 메커니즘.
3. **Swish Activation**: ReLU보다 매끄러운 특성으로 심층 신경망 학습 시 기울기 소실 문제를 완화.

> **cf. 용어 해설**
> - **MBConv (Mobile Inverted Bottleneck Convolution)**: MobileNet-V2에서 도입된 효율적 합성곱 블록. 채널을 확장→깊이별 합성곱→축소하는 역병목(Inverted Bottleneck) 구조.
> - **Inverted Residual**: 기존 Bottleneck(축소→연산→확장)과 반대로, 저차원 입력을 고차원으로 확장한 뒤 경량 합성곱을 수행하고 다시 저차원으로 압축하는 구조.
> - **Squeeze-and-Excitation (SE)**: 채널별 중요도를 적응적으로 재조정(Recalibration)하는 모듈. 글로벌 평균 풀링으로 채널 정보를 압축(Squeeze)한 뒤 FC 층으로 가중치를 생성(Excitation)하여 중요 채널을 강조함.
> - **Attention (주의 집중)**: 입력의 특정 부분에 선택적으로 집중하여 중요한 정보에 더 큰 가중치를 부여하는 메커니즘. SE 블록은 채널 차원의 Attention에 해당.
> - **채널(Channel)**: 특징 맵(Feature Map)의 깊이 방향 차원. RGB 이미지는 3채널이며, 합성곱 층의 출력 채널 수는 필터 개수에 대응.
> - **Swish Activation**: f(x) = x · σ(x) (σ는 시그모이드)로 정의되는 활성화 함수. ReLU와 달리 음수 영역에서도 약간의 기울기를 가져 학습이 더 안정적.
> - **ReLU**: Rectified Linear Unit. f(x) = max(0, x)로 정의되는 대표적 활성화 함수. 구조가 단순하나 음수 영역에서 기울기가 0이 되는 한계가 있음.
> - **기울기 소실(Vanishing Gradient)**: 심층 신경망에서 역전파 시 기울기(Gradient)가 층을 거칠수록 급격히 작아져 앞쪽 층의 가중치가 거의 업데이트되지 않는 문제.
> - **Bottleneck**: 신경망에서 채널 수를 의도적으로 줄여 연산량을 절감하는 병목 구조. MBConv에서는 역으로(Inverted) 확장 후 축소하는 방식을 사용.

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

> **cf. 용어 해설**
> - **MobileNet-V2**: Google에서 제안한 경량 CNN. Inverted Residual과 Linear Bottleneck을 도입하여 모바일 환경에 최적화된 구조.
> - **ResNet-50**: 50층 깊이의 잔차 신경망(Residual Network). Skip Connection으로 심층 학습을 가능하게 한 산업계 표준 모델.
> - **ConvNeXt**: Transformer의 설계 원리를 CNN에 적용하여 현대화한 차세대 합성곱 신경망. 높은 정확도를 보이나 연산량이 큼.
> - **ViT-Base**: Vision Transformer의 기본(Base) 크기 변형. 86M 파라미터로 대규모 데이터셋에서 강력한 성능을 보이나 소규모 데이터에서는 비효율적.
> - **Swin Transformer**: Shifted Window 기반의 계층적 Transformer. 지역적 윈도우 내 Self-Attention으로 연산량을 줄이고, 윈도우를 이동(Shift)하여 전역 정보를 교환.
> - **Parameters**: 모델의 학습 가능한 가중치 총 수. 모델 크기 및 메모리 요구량의 지표.
> - **FLOPS**: 한 번의 순전파(Forward Pass)에 필요한 부동소수점 연산 횟수. 추론 속도의 간접 지표.
> - **Top-1**: 모델의 최상위 1개 예측이 정답인 비율. ImageNet 벤치마크의 표준 정확도 지표.

### 4.1 최신 대형 모델 미채택 사유

#### 실시간성 제약 (Latency)
- **InternViT / CoCa**: 수십억 파라미터의 파운데이션 모델로, 서버급 GPU가 필요. 실시간 Tact Time(평균 100ms 이내)을 맞출 수 없음.
- **Swin Transformer / ViT**: Transformer 기반은 전역 특징 포착에 강하지만, 이물(ROI) 분류 같은 **국부적 텍스처 인식**에는 CNN의 귀납적 편향(Inductive Bias)이 적은 데이터에서 더 효과적.

#### 데이터 효율성
- ViT 계열은 충분한 성능을 내기 위해 대규모 데이터가 필요. 산업 현장에서 이물 이미지는 수집이 어렵고(불균형 데이터), 증강에 의존해야 하므로 EfficientNet의 효율성이 더 높음.

#### 배포 편의성
- EfficientNet-B0는 ONNX, OpenVINO, TensorRT에서 가장 잘 지원되는 검증된 모델. CPU 환경에서도 원활한 추론 가능.

> **cf. 용어 해설**
> - **InternViT**: 대규모 멀티모달 파운데이션 모델에 사용되는 초대형 Vision Transformer. 수십억 파라미터로 범용 시각 표현을 학습하나 엣지 배포에 부적합.
> - **CoCa (Contrastive Captioners)**: Google에서 제안한 대조 학습(Contrastive Learning)과 캡셔닝을 결합한 대규모 멀티모달 파운데이션 모델.
> - **파운데이션 모델(Foundation Model)**: 대규모 데이터로 사전 학습되어 다양한 하류 작업(Downstream Task)에 범용적으로 활용할 수 있는 초대형 모델.
> - **Swin Transformer**: Shifted Window 기반 계층적 Vision Transformer. 윈도우 단위 Self-Attention으로 연산 효율을 높인 구조.
> - **ViT (Vision Transformer)**: 이미지를 패치 시퀀스로 변환하여 Transformer 인코더에 입력하는 영상 분류 모델. 전역적 관계 학습에 강점.
> - **Transformer**: Self-Attention 메커니즘을 핵심으로 하는 신경망 구조. 원래 자연어 처리를 위해 제안되었으며, 이후 영상 분야로 확장됨.
> - **귀납적 편향(Inductive Bias)**: 모델 구조에 내재된 사전 가정. CNN은 "인접 픽셀이 관련 있다"는 지역성(Locality) 편향이 있어 적은 데이터에서도 효과적으로 학습 가능.
> - **CNN (Convolutional Neural Network)**: 합성곱 신경망. 지역적 수용장(Receptive Field) 기반으로 이미지 특징을 계층적으로 추출하는 구조.
> - **국부적 텍스처(Local Texture)**: 이미지의 작은 영역에서 관찰되는 반복적 패턴이나 표면 질감. 이물 검사에서 미세 이물의 외관 특징에 해당.
> - **데이터 효율성(Data Efficiency)**: 적은 양의 학습 데이터로도 높은 성능에 도달할 수 있는 모델의 능력. 산업 현장에서는 데이터 수집이 어려워 데이터 효율성이 중요.
> - **불균형 데이터(Imbalanced Data)**: 클래스 간 샘플 수가 크게 차이나는 데이터셋. 이물 검사에서는 정상(양품)이 대다수이고 이물(불량)이 극소수인 불균형이 발생.
> - **증강(Augmentation)**: 학습 데이터에 회전, 반전, 밝기 조절 등의 변환을 적용하여 데이터의 다양성과 양을 인위적으로 늘리는 기법.
> - **배포(Deployment)**: 학습 완료된 모델을 실제 운영 환경(서버, 엣지 디바이스 등)에 탑재하여 서비스하는 과정.

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

> **cf. 용어 해설**
> - **EfficientNet**: Google Brain에서 제안한 CNN 아키텍처. 복합 스케일링(Compound Scaling)을 통해 연산 효율 대비 최고 정확도를 달성한 모델 패밀리.
> - **RepViT**: 재파라미터화(Reparameterization) 기법을 Vision Transformer에 적용하여 추론 시 구조를 단순화한 경량 하이브리드 모델.
> - **MobileOne**: Apple에서 발표한 모바일 최적화 모델. 학습 시 다중 분기를 사용하고 추론 시 재파라미터화로 단일 분기로 병합.
> - **ConvNeXt**: Transformer의 설계 원칙(큰 커널, LayerNorm, GELU 등)을 순수 CNN에 적용하여 현대화한 차세대 합성곱 신경망.
> - **ViT (Vision Transformer)**: 이미지를 16×16 등 고정 크기 패치로 나누어 Transformer 인코더에 입력하는 비전 모델. 대규모 데이터에서 뛰어난 성능.
> - **MBConv (Mobile Inverted Bottleneck Convolution)**: 확장→깊이별 합성곱→압축의 역병목 구조로 구성된 효율적 합성곱 블록. EfficientNet의 기본 빌딩 블록.
> - **timm (PyTorch Image Models)**: Hugging Face에서 관리하는 오픈소스 라이브러리. 700개 이상의 사전 학습 모델과 다양한 학습 유틸리티를 제공.

---

## 6. 결론 (Conclusion)

EfficientNet-B0는 **"최소한의 자원으로 최대한의 정확도"**를 내야 하는 실시간 이물 검사 시스템에 가장 안정적인 선택이다.

1. **ResNet50 대비**: 정확도는 높이면서 연산량은 1/10 수준으로 절감.
2. **MobileNet 대비**: 연산량 차이는 미미하나, 고수준 특징 추출 능력으로 분류 정확도 대폭 향상.
3. **최신 ViT 대비**: 적은 학습 데이터로도 안정적인 수렴이 가능하며 국소적 특징(이물) 포착에 유리.

더 높은 속도가 필요한 경우 **RepViT-M0.9** (정확도+속도 균형) 또는 **MobileOne-S1** (최고속)을 선택할 수 있으며, UI의 학습 탭에서 모델 아키텍처를 변경하면 된다.

> **향후 계획**: 하드웨어 업그레이드나 대규모 데이터 확보 시, EfficientNet-V2 또는 EfficientFormer 등의 도입을 검토할 예정.

> **cf. 용어 해설**
> - **연산량**: 모델이 한 번의 추론(또는 학습)을 수행하는 데 필요한 총 부동소수점 연산 횟수(FLOPS). 모델의 계산 비용과 속도를 결정하는 핵심 지표.
> - **특징 추출(Feature Extraction)**: 입력 이미지로부터 분류에 유용한 정보(에지, 텍스처, 형태 등)를 계층적으로 추출하는 과정. Backbone 네트워크의 주요 역할.
> - **수렴(Convergence)**: 학습 과정에서 손실(Loss) 값이 점차 안정적인 최솟값에 도달하는 상태. 적은 데이터로도 빠르게 수렴하는 모델이 산업 현장에서 유리.
> - **EfficientNet-V2**: EfficientNet의 후속 버전. 학습 효율성을 개선하기 위해 점진적 학습(Progressive Training)과 Fused-MBConv 블록을 도입한 모델.
> - **EfficientFormer**: Transformer 구조를 모바일/엣지 디바이스에서 효율적으로 실행할 수 있도록 경량화한 모델. 하드웨어 친화적 설계로 빠른 추론 속도를 달성.
