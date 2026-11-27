# Smart Kitchen Assistant

**온디바이스 AI 기반 실시간 연기 감지 및 Hands-Free 레시피·타이머 제어 시스템**

| 기간 | 인원 |
|---|---|
| 2026.07.29 ~ 2026.08.03 | 2인 팀 |

## 1. 프로젝트 개요

Smart Kitchen Assistant는 조리 중 발생하는 연기를 카메라 영상에서 감지하고 화면과 소리로 알려주는 주방 보조 앱입니다. 손에 식재료나 양념이 묻어 화면을 터치하기 어려울 때는 손동작으로 레시피·YouTube 화면과 화구별 타이머를 조작할 수 있습니다. 화면을 보기 어려운 상황에서는 타이머의 남은 시간을 음성으로 확인할 수 있습니다.

### 팀과 역할

| 이름 | 역할 |
|---|---|
| Gabin (`dev-gabin`) | PySide6 메인 대시보드·미니 모드 구현, YOLOv8 연기 감지 모델 학습·성능 검증, 화면 전환 오류 수정 및 조작 흐름 개선, 발표 자료 제작·시연 화면 구성 |
| WhatTheHJ (hyeonju) | MediaPipe Hands·OpenCV KNN 기반 손동작 인식, 제스처를 통한 레시피·YouTube 화면과 화구별 타이머 제어, 음성 명령을 통한 타이머 남은 시간 안내 구현 |

프로젝트 기획, 문제 정의, 기능 통합과 오류 해결은 함께 진행했습니다.
## 2. 시스템 구성

**웹캠 → OpenCV 영상 수집**

- **연기 감지** → YOLOv8 → 대시보드 화면 경고와 PC 경고음
- **손동작 인식** → MediaPipe Hands + OpenCV KNN → 화구 선택·타이머 조작 또는 레시피·YouTube 제어

**마이크 → Google 음성 인식 → 대시보드의 화구별 타이머 조회 → 남은 시간 음성 안내**

연기 감지와 손동작 분류는 Windows PC에서 처리합니다. 음성 명령 인식에는 인터넷 연결이 필요합니다.

## 3. 사용 기술

| 분야 | 기술 |
|---|---|
| 영상·연기 감지 | Python, OpenCV, Ultralytics YOLOv8, PyTorch |
| 손동작 인식·제어 | MediaPipe Hands, OpenCV KNN, PyAutoGUI |
| 화면·경보 | PySide6, Windows `winsound` |
| 음성 입력·출력 | SpeechRecognition, PyAudio, Google 음성 인식, pyttsx3 |
| 개발 도구 | VS Code, Git, GitHub |

## 4. 데이터와 모델

- **연기 감지** — 연기 이미지로 YOLOv8 모델을 추가 학습했습니다. 학습 스크립트는 Roboflow 형식의 데이터 설정을 사용하며, 입력 크기 640과 50 epoch로 설정되어 있습니다. 데이터셋 이름, 라이선스, 이미지 수와 학습·검증 분할은 확인이 필요합니다.
- **손동작 분류** — 손 특징과 분류 라벨을 담은 CSV를 OpenCV KNN 학습에 사용했습니다. 확인한 CSV에는 헤더를 제외한 데이터 944행이 있으며, 수집 조건과 학습·검증 분할은 확인이 필요합니다.

원본 연기 이미지는 로컬의 `data/raw_smoke_dataset/`에서 관리했습니다. 원본 이미지, 제스처 학습 CSV와 모델 가중치는 공개 저장소에 포함하지 않습니다.

## 5. 동작 방식

1. **연기 감지** — OpenCV로 웹캠 프레임을 받아 YOLOv8로 연기를 탐지합니다. 탐지 조건을 충족하면 대시보드에 위험 상태를 표시하고 PC 스피커로 경고음을 냅니다.
2. **손동작 제어** — MediaPipe Hands가 추출한 손 좌표를 바탕으로 OpenCV KNN이 제스처를 분류합니다. 결과를 화구 선택·타이머 조작에 연결하고, 레시피·YouTube의 재생·일시정지·스크롤에는 PyAutoGUI를 사용합니다.
3. **타이머 음성 안내** — 음성 명령에서 요청한 화구를 확인하고, 타이머의 남은 시간을 `pyttsx3`로 읽어줍니다.

## 6. 실행 방법

1. Windows에서 Python 가상환경을 활성화한 뒤, 프로젝트 루트에서 `python -m pip install -r requirements.txt`로 의존성을 설치합니다.
2. 사용 권한을 확인한 연기 감지 가중치를 `models/custom_smoke_best_v4.pt`에 배치합니다.
3. 코드와 호환되는 제스처 학습 CSV를 `data/gesture_train.csv`에 배치합니다.
4. 웹캠과 마이크를 연결하고 프로젝트 루트에서 `python main.py`로 실행합니다.

### 프로젝트 구조

```text
Project-smart-kitchen-assistant/
├── .vscode/
│   └── settings.json              # VS Code 프로젝트 설정
├── assets/
│   └── icons/                     # 버튼 및 화면 아이콘
├── src/
│   ├── burner/
│   │   ├── burner_detector.py     # 화구 영역 감지
│   │   └── smoke_detector.py      # YOLOv8 연기 감지
│   ├── gesture/
│   │   ├── controller.py          # 손동작 인식 및 제어
│   │   ├── voice_detector.py      # 음성 명령 및 타이머 안내
│   │   ├── web_control.py         # 웹 브라우저 제어
│   │   └── youtube_controller.py  # YouTube 제어
│   └── kitchen_ui_overlay.py      # 대시보드 및 미니 모드
├── .gitignore
├── README.md
├── main.py                        # 앱 실행 진입점
├── requirements.txt               # Python 의존성
└── train_smoke_model.py           # 연기 감지 모델 학습
```

학습 데이터와 가중치는 각각 로컬의 `data/`, `models/`에 준비합니다. 개발 과정에서 사용한 모델은 기본 가중치 `yolov8n.pt`와 연기 감지 실험 가중치 `custom_smoke_best.pt`, `custom_smoke_best_v2.pt`, `custom_smoke_best_v3.pt`, `custom_smoke_best_v4.pt`입니다.

## 7. 성능 비교

| YOLOv8 연기 감지 평가 지표 | 결과 |

<img width="2400" height="1200" alt="image" src="https://github.com/user-attachments/assets/4de8d4c6-20ef-4666-bfaf-3a52f4b22929" />

|---|---:|
| 정밀도 (Precision) | 0.87 |
| 재현율 (Recall) | 0.83 |
| mAP50 | 0.91 |
| mAP50–95 | 0.63 |

모델 평가 결과입니다. 최적화 전후 수치, 평가에 사용한 가중치 버전과 검증 데이터 분할, FPS와 경고 지연 시간은 확인이 필요합니다. [모델 평가 그래프](https://app.notion.com/p/gabin-dev/Gabin-Portfolio-3bf06eb7616180c983ddde0fd40d9abc)

## 8. 결과
<img width="1047" height="778" alt="image" src="https://github.com/user-attachments/assets/64617b5e-55dd-480c-ab14-c75ad8d1ba8e" />


*메인 대시보드 화면. 왼쪽에는 카메라 뷰, 오른쪽에는 화구별 타이머 4개, 하단에는 제스처 인식·선택 화구·연기 감지 상태를 배치했습니다.*

PySide6 메인 대시보드와 미니 모드에 웹캠 영상, 연기 감지 경고, 레시피 화면 조작과 화구별 독립 타이머를 통합했습니다. 손동작으로 화면과 타이머를 조작하고, 음성으로 남은 시간을 확인할 수 있습니다. 카메라와 화면이 있는 주방 디스플레이에도 이 기능을 통합할 수 있습니다.

## 9. 시행착오와 해결

| 문제 | 원인 | 해결 |
|---|---|---|
| 같은 손동작을 유지해도 실행 환경에 따라 판정 시점이 달라짐 | 같은 프레임 수라도 FPS에 따라 실제 경과 시간이 달라짐 | 실제 경과 시간을 기준으로 유지 시간을 판정 |
| 개발 환경마다 OpenCV 의존성 선언이 달랐음 | `opencv-python`과 `opencv-contrib-python`이 혼재함 | `requirements.txt`의 선언을 `opencv-contrib-python`으로 정리 |
| 화면 전환 중 조작 흐름이 끊김 | 메인 대시보드와 미니 모드 전환 중 오류가 발생함 | 화면 전환 동작을 수정하고 조작 흐름을 개선 |

## 10. 배운 점

YOLOv8 연기 감지 모델을 추가 학습하고 정밀도·재현율·mAP를 비교하면서, 각 지표가 모델의 탐지 성능을 어떻게 보여주는지 배웠습니다. 또한 탐지 결과를 PySide6 신호로 받아 화면 경고와 경고음에 연결하며, 모델이 감지한 상황을 사용자가 바로 알아차릴 수 있도록 전달하는 방법을 익혔습니다. 메인 대시보드와 미니 모드의 전환 오류를 수정하면서, 각 화면의 기능뿐 아니라 화면을 오갈 때의 동작도 확인해야 한다는 점을 배웠습니다.
