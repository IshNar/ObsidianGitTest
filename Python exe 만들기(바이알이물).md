## 1. CPU 전용 가상환경 만들기 (.venv_cpu)

프로젝트 루트에서 터미널(파워셸/명령 프롬프트) 열고:

# 1) 새 venv 생성

python -m venv .venv_cpu

# 2) 새 venv 활성화 (PowerShell 기준)

.\.venv_cpu\Scripts\Activate

> 앞으로 exe 빌드는 항상 이 .venv_cpu 안에서만 진행합니다.

> (기존 .venv는 개발/디버깅/GPU용으로 그대로 두세요.)

---

## 2. CPU 전용 PyTorch + 나머지 의존성 설치

같은 터미널(.venv_cpu 활성화 상태)에서:

# 1) CPU 전용 PyTorch / torchvision 설치

pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu

# 2) 나머지 라이브러리 설치 (requirements.txt 재사용)

pip install -r requirements.txt

- 이렇게 하면 CUDA 없는 CPU 전용 torch가 설치됩니다.

- 용량이 좀 크고 설치 시간이 걸릴 수 있습니다.

---

## 3. CPU venv에서 프로그램 테스트

여전히 .venv_cpu 활성화 상태에서:

python src/main.py

- UI가 뜨면:

- 모델 로드 버튼으로 classification_model.pth를 선택해보세요.

- 또는 Classification 탭에서 학습/저장한 모델을 로드해도 됩니다.

- 이 환경에서 모델 로드 + 분류가 정상 동작하는지 먼저 확인합니다.

(터미널에 [DL Classifier] 모델 로드 완료: ... 로그가 찍히는지 확인)

> 여기까지 통과해야 CPU 전용 exe도 같은 동작을 기대할 수 있습니다.

---

## 4. CPU venv에서 exe 빌드

이제 .venv_cpu 안에서 exe를 만듭니다:

python build_exe.py

- 시간이 꽤 걸릴 수 있습니다.

- 끝나면 터미널에:

  완료. 실행 파일: dist\ForeignBodyInsp\ForeignBodyInsp.exe

- dist\ForeignBodyInsp\ 폴더가 생성됩니다.

---

## 5. exe용 모델 파일 배치 & 실행

1. dist\ForeignBodyInsp\ 폴더 안에 classification_model.pth 를 복사합니다.

- 예:

F:\PythonProjects\ForeignBodyInsp-vial-inspection-system\dist\ForeignBodyInsp\classification_model.pth

1. ForeignBodyInsp.exe 더블클릭으로 실행

- 우리가 코드에 넣어둔 대로, exe로 실행될 때

동일 폴더의 classification_model.pth를 자동으로 로드 시도합니다.

- 자동 로드가 되면 Use DeepLearning 체크가 켜지고,

모델 이름/클래스가 로그에 출력됩니다.

1. 자동 로드가 안 되면:

- exe에서 “모델 로드” 버튼을 눌러 같은 폴더의 classification_model.pth를 직접 선택해 보세요.

- 실패 시 이제는 구체적인 에러 메시지가 팝업으로 나오니, 그 문구를 알려주시면 추가로 봐 드리겠습니다.

---

## 6. 정리: 환경별 역할

- 기존 .venv (GPU 포함)

- 개발 / 디버깅 / F5 실행 / 실험용

- python src/main.py 또는 VS Code F5

- 새 .venv_cpu + 빌드된 exe

- 배포 / 현장 PC / Python 없는 환경에서 사용

- dist\ForeignBodyInsp\ForeignBodyInsp.exe + 같은 폴더의 classification_model.pth

여기까지 따라 하신 뒤에도 exe에서 모델 로드 오류가 나오면,

그때 뜨는 오류 메시지 전체를 한 번 더 공유해 주세요.

지금은 CPU 전용 + 전체 torch 번들을 쓰고 있어서,

남는 문제는 거의 “DLL 누락”이나 “환경 변수” 레벨일 가능성이 큽니다.