# 알약 객체 검출 개인 프로젝트

YOLOv8n 사전학습 모델로 이미지 속 알약의 클래스와 위치를 검출합니다. AI 도구의 코드 작성 및 오류 대응 지원을 받아 Google Colab에서 실행했습니다.

## 결과
| 항목 | 값 |
|---|---:|
| 훈련용 이미지 / JSON | 232 / 763 |
| 클래스 | 56 |
| 학습 / 검증 이미지 | 186 / 46 |
| 내부 검증 Precision / Recall | 0.767 / 0.590 |
| 내부 검증 mAP@50 / mAP@50-95 | 0.742 / 0.711 |
| Kaggle mAP@[0.75:0.95] | 0.10093 |
| 테스트 이미지 / 예측 객체 | 287 / 1,148 |

Kaggle 점수는 제출 후 확인한 사용자 제공 수치입니다. 내부 수치는 best.pt 최종 검증 로그 기준으로 반올림했습니다. 마지막 epoch 수치와는 다를 수 있습니다.

## 실행 방법
1. `pill_detection.ipynb`를 Google Colab에서 엽니다.
2. 런타임에서 T4 GPU를 선택합니다.
3. 제공된 데이터를 Google Drive에 업로드하고 첫 셀의 `ROOT`를 수정합니다.
4. 셀을 위에서 아래로 실행하고 Drive 연결을 허용합니다.
5. 마지막 셀에서 생성되는 `submission.csv`를 Kaggle에 제출합니다.

입력 구조:
```
sprint_ai_project1_data/
  train_images/       # PNG (하위 폴더 허용)
  train_annotations/  # COCO JSON (하위 폴더 허용)
  test_images/        # 숫자 파일명 PNG
```
기존 실행 당시 Python 3.13.15 / torch 2.11.0+cu130 / ultralytics 8.4.184 / Tesla T4. Colab 기본 PyTorch와 아래 requirements를 사용합니다. GPU 환경에 따라 수치가 조금 달라질 수 있습니다.

## 처리 방식
이미지별로 분리된 JSON 주석 병합 → 비정상 박스 1개 제외 → 경계 좌표 보정 → 연속 클래스 번호 변환 → seed 42로 이미지 단위 무작위 분할 → YOLOv8n 15 epochs 학습 → 원본 클래스 번호 복원 후 CSV 생성. 이미지 크기 640, batch 16, 추론 conf 0.01, NMS IoU 0.5, 최대 검출 4개입니다.

## 한계
- 검증은 46장, 150개 객체이며 모든 56클래스를 포함하지 않습니다.
- 촬영 조건만 다른 유사 이미지가 학습·검증에 나뉘었을 가능성이 있습니다. 그룹 분할로 검증하지 않았습니다.
- 내부 mAP@50-95와 대회 mAP@[0.75:0.95]는 직접 비교할 수 없습니다.
- 안내상 테스트에 학습 미포함 클래스가 존재하며 현재 모델은 56개 학습 클래스만 출력합니다.
- 추가 데이터, 추가 모델 비교, 하이퍼파라미터 탐색, 배포는 수행하지 않았습니다.

## 제출 문서
- [분석 보고서 PDF](report.pdf)
- [업무일지 PDF](worklog.pdf)
- [업무일지 원문](worklog.md)

원본 데이터와 학습 가중치, submission.csv는 이 묶음에 포함하지 않았습니다. 실행 결과는 Drive의 `pill_results`에 보관됩니다. 별도 재학습 없이 모델을 쓰려면 본인 Drive의 baseline/weights/best.pt를 사용합니다. 금지된 AI Hub 추가 데이터는 사용하지 않았습니다.

## 참고
- 대회: https://www.kaggle.com/competitions/ai15-level1-project
- Ultralytics: https://docs.ultralytics.com/modes/train/
