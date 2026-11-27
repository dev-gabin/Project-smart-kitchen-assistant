# Smart Kitchen Assistant

**온디바이스 AI 기반 실시간 연기 감지 및 Hands-Free 레시피·타이머 제어 시스템**

| 기간 | 인원 |
|---|---|
| 2026.07.29 ~ 2026.08.03 | 2인 팀 |

## 1. 프로젝트 개요

Smart Kitchen Assistant는 조리 중 발생하는 연기를 카메라 영상에서 감지하고 화면과 소리로 알려주는 주방 보조 앱입니다. 손에 식재료나 양념이 묻어 화면을 터치하기 어려울 때는 손동작으로 레시피·YouTube 화면과 화구별 타이머를 조작할 수 있습니다. 화면을 보기 어려울 때는 음성으로 타이머의 남은 시간을 확인할 수 있습니다.

### 팀과 역할

| 이름 | 역할 |
|---|---|
| Gabin (`dev-gabin`) | **AI 비전** — YOLOv8 연기 감지 모델 학습과 성능 검증<br>**UI·경고 연동** — PySide6 메인 대시보드와 미니 모드 구현, 연기 감지 결과를 화면 경고와 경고음에 연결, 화면 전환 오류 수정과 조작 흐름 개선<br>**발표·시연** — 발표자료 제작과 시연 화면 구성 |
| WhatTheHJ (hyeonju) | **손동작 인식·제어** — MediaPipe Hands와 OpenCV KNN 기반 제스처 분류, 레시피·YouTube 화면과 화구별 타이머 제어<br>**음성 인터페이스** — 음성 명령으로 타이머의 남은 시간을 안내하는 기능 구현 |

프로젝트 기획, 문제 정의, 기능 통합과 오류 해결은 함께 진행했습니다.

## 2. 시스템 구성

| 구성 | 연결과 역할 |
|---|---|
| 웹캠 → OpenCV → YOLOv8 | 웹캠 프레임을 받아 연기를 탐지 |
| YOLOv8 → PySide6 대시보드·`winsound` | 탐지 결과를 화면의 위험 상태 표시와 PC 경고음으로 전달 |
| 웹캠 → OpenCV → MediaPipe Hands → OpenCV KNN | 영상에서 손 좌표를 추출하고 제스처를 분류 |
| 제스처 분류 결과 → 대시보드·PyAutoGUI | 대시보드의 화구 선택·타이머 조작과 레시피·YouTube 화면 제어에 분류 결과를 연결 |
| 마이크 → Google 음성 인식 → 화구별 타이머 → `pyttsx3` | 음성 명령에서 요청한 화구를 확인하고 남은 시간을 읽어줌 |

연기 감지와 손동작 분류는 Windows PC에서 처리합니다. Google 음성 인식을 사용하려면 인터넷 연결이 필요합니다.

## 3. 사용 기술

| 분야 | 기술 |
|---|---|
| 영상·연기 감지 | Python, OpenCV, Ultralytics YOLOv8, PyTorch |
| 손동작 인식·제어 | MediaPipe Hands, OpenCV KNN, PyAutoGUI |
| 화면·경보 | PySide6, Windows `winsound` |
| 음성 입력·출력 | SpeechRecognition, PyAudio, Google 음성 인식, pyttsx3 |
| 개발 도구 | VS Code, Git, GitHub |

## 4. 데이터와 모델

- **연기 감지** — 연기 이미지로 YOLOv8 모델을 추가 학습했습니다. 학습 스크립트는 Roboflow 형식의 데이터 설정을 사용하며, 입력 크기는 640픽셀, 학습 설정은 50 epoch입니다. 데이터셋 이름, 라이선스, 이미지 수와 학습·검증 분할은 확인이 필요합니다.
- **손동작 분류** — 손 특징과 분류 라벨을 담은 CSV를 OpenCV KNN 학습에 사용했습니다. 확인한 CSV에는 헤더를 제외하고 데이터 944행이 있습니다. 수집 조건과 학습·검증 분할은 확인이 필요합니다.

원본 연기 이미지는 로컬의 `data/raw_smoke_dataset/`에서 관리했습니다. 원본 이미지, 제스처 학습 CSV와 모델 가중치는 공개 저장소에 포함하지 않습니다.

## 5. 동작 방식

1. **연기 감지** — 조리 중 웹캠 영상에서 탐지 조건을 충족하는 연기가 감지되면 대시보드에 위험 상태가 표시되고 PC 스피커에서 경고음이 납니다.
2. **손동작으로 타이머 조작** — 사용자가 지정된 손동작을 하면 앱이 제스처를 분류해 화구 선택이나 타이머 조작으로 연결합니다. 대시보드에서는 선택한 화구와 타이머 상태를 확인할 수 있습니다.
3. **손동작으로 레시피 화면 조작** — 레시피나 YouTube를 보는 중에는 제스처로 재생·일시정지·스크롤을 제어할 수 있습니다.
4. **타이머 음성 안내** — 사용자가 음성으로 화구의 남은 시간을 물으면 앱이 해당 화구의 타이머를 조회해 남은 시간을 읽어줍니다.

## 6. 실행 방법

1. Windows에서 Python 가상환경을 활성화하고, 프로젝트 루트에서 의존성을 설치합니다.

   ```powershell
   python -m pip install -r requirements.txt
   ```

2. 연기 감지 모델 가중치를 `models/custom_smoke_best_v4.pt`에 배치합니다.
3. 코드와 호환되는 제스처 학습 CSV를 `data/gesture_train.csv`에 배치합니다.
4. 웹캠과 마이크를 연결하고 프로젝트 루트에서 앱을 실행합니다.

   ```powershell
   python main.py
   ```

### 프로젝트 구조

```text
Project-smart-kitchen-assistant/
├── .vscode/
│   └── settings.json              # VS Code 프로젝트 설정
├── assets/
│   └── icons/                     # 버튼 및 화면 아이콘
├── src/
│   ├── burner/
│   │   ├── burner_detector.py      # 화구 영역 감지
│   │   └── smoke_detector.py       # YOLOv8 연기 감지
│   ├── gesture/
│   │   ├── controller.py           # 손동작 인식 및 제어
│   │   ├── voice_detector.py       # 음성 명령 및 타이머 안내
│   │   ├── web_control.py          # 웹 브라우저 제어
│   │   └── youtube_controller.py   # YouTube 제어
│   └── kitchen_ui_overlay.py       # 대시보드 및 미니 모드
├── .gitignore
├── README.md
├── main.py                         # 앱 실행 진입점
├── requirements.txt                # Python 의존성
└── train_smoke_model.py            # 연기 감지 모델 학습
```

개발 과정에서는 기본 가중치 `yolov8n.pt`와 연기 감지 실험 가중치 `custom_smoke_best.pt`, `custom_smoke_best_v2.pt`, `custom_smoke_best_v3.pt`, `custom_smoke_best_v4.pt`를 사용했습니다.

## 7. 모델 성능 평가

프로젝트 자료에 기록된 YOLOv8 연기 감지 모델의 평가 결과입니다.

| 정밀도 (Precision) | 재현율 (Recall) | mAP@50 | mAP@50–95 |
|---:|---:|---:|---:|
| 0.87 | 0.83 | 0.91 | 0.63 |

<img width="2400" height="1200" alt="YOLOv8 연기 감지 모델 평가 그래프" src="https://github.com/user-attachments/assets/4de8d4c6-20ef-4666-bfaf-3a52f4b22929" />

위 수치에 대응하는 가중치 버전과 검증 데이터 분할, 앱 실행 FPS와 경고 지연 시간은 확인이 필요합니다. [모델 평가 그래프](https://app.notion.com/p/gabin-dev/Gabin-Portfolio-3bf06eb7616180c983ddde0fd40d9abc)

## 8. 결과

### 메인 대시보드

<img width="1047" height="778" alt="Smart Kitchen Assistant 메인 대시보드" src="https://github.com/user-attachments/assets/64617b5e-55dd-480c-ab14-c75ad8d1ba8e" />

왼쪽에는 카메라 영상, 오른쪽에는 화구별 타이머 4개가 있습니다. 하단에서는 제스처 인식 상태, 선택한 화구, 연기 감지 상태를 확인할 수 있습니다.

### 조리 중 조작

PySide6 메인 대시보드와 미니 모드를 구현했습니다. 사용자는 조리하면서 손동작으로 레시피 화면이나 화구별 타이머를 조작하고, 화면을 보기 어려울 때는 음성으로 남은 시간을 확인할 수 있습니다. 연기가 감지되면 대시보드에 위험 상태를 표시하고 경고음을 재생합니다.

## 9. 시행착오와 해결

| 문제 | 원인 | 해결 |
|---|---|---|
| 같은 손동작을 같은 시간 동안 유지해도 실행 환경에 따라 판정 시점이 달라졌음 | 유지 조건을 프레임 수로 계산하면 FPS가 달라질 때 같은 프레임 수에 해당하는 실제 시간도 달라짐 | 프레임 개수 대신 실제 경과 시간을 기준으로 제스처 유지 여부를 판정 |
| 개발 환경마다 설치할 OpenCV 패키지가 달랐음 | 의존성에 `opencv-python`과 `opencv-contrib-python`이 혼재했음 | `requirements.txt`의 OpenCV 의존성을 `opencv-contrib-python`으로 정리 |
| 메인 대시보드와 미니 모드 사이를 오갈 때 조작 흐름이 끊겼음 | 두 화면을 전환하는 과정에서 오류가 발생했음 | 화면 전환 동작을 수정하고, 두 모드를 오가는 조작 흐름을 개선 |

## 10. 배운 점

### AI 비전

YOLOv8 연기 감지 모델을 추가 학습하고 정밀도·재현율·mAP를 비교했습니다. 정밀도로는 연기라고 탐지한 결과가 얼마나 맞았는지, 재현율로는 실제 연기를 얼마나 잘 찾아냈는지 확인하는 방법을 배웠습니다. mAP@50과 mAP@50–95를 함께 보면서 탐지 박스의 위치를 더 엄격하게 평가했을 때 성능이 어떻게 달라지는지도 살펴봤습니다.

### 실시간 경고 연동

연기 탐지 결과를 PySide6 신호로 받아 대시보드의 위험 상태 표시와 PC 경고음에 연결했습니다. 사용자가 화면을 볼 때는 상태 표시로, 다른 곳을 보고 있을 때는 경고음으로 상황을 알아차릴 수 있도록 모델의 출력을 알림으로 연결하는 방법을 익혔습니다.

### UI 전환과 검증

메인 대시보드와 미니 모드 사이의 전환 오류를 수정했습니다. 각 화면의 기능을 개별적으로 확인하는 것과 함께, 사용자가 화면을 오갈 때도 조작을 이어갈 수 있는지 확인해야 한다는 점을 배웠습니다.
