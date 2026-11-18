# 1_Shrink 메모

Shrink: 한 스텝 비용을 줄임  
논문 목록과 PDF: [`../Related Work/Efficient KD papers/1_Shrink/`](<../Related Work/Efficient KD papers/1_Shrink>)  
작성 규칙과 항목 설명: [README](README.md)

담당: 

아래 항목은 원문·공식 코드·공개 결과에서 확인한 내용이며, 이 정리를 위해 직접 재실험한 결과는 아님.

「잃는 것」은 **[실측]**(조건이 명시된 관찰 결과)과 **[구조]**(방법 정의·공식 코드에서 확인되는 적용 제약)를 구분함. 구조상 제약을 능력 저하의 실험 증거로 해석하지 않음. 근거가 있는 내용만 짧게 적으며, 해당 내용이 없으면 항목을 생략함.

### 01. MaskedKD (The Role of Masking for Efficient Supervised KD of ViTs) ([arXiv: 2302.10494](https://arxiv.org/abs/2302.10494))
- 효율화 방식: 입력 토큰을 마스킹해 교사의 어텐션·선형 변환·MLP 계산량을 줄임. 학생의 마지막 어텐션 층을 활용해 교사에게 입력할 토큰을 선택함.
- 얻는 것: 교사의 지도 비용 감소. 교사가 처리하는 토큰에 필요한 계산을 줄임.
- 잃는 것:
  - [실측·외부] Waterbirds에서 전체 입력 교사는 맞혔지만 50% 패치 입력 교사는 틀린 샘플 2.54~3.59%(3개 학생 seed의 마스크). 반대로 마스킹 후 맞힌 샘플 1.33~1.90%. 학생 능력 손실 자체를 측정한 것은 아님. [CSV](https://github.com/jeehoo0507/kshs-aimlab-benchmarks/blob/main/results/waterbirds/mask_test.csv)
  - [구조] 토큰 선택에 학생의 마지막 층 어텐션이 필요함. 로짓만 반환하는 학생으로는 이 선택 규칙을 그대로 실행할 수 없음. [공식 코드](https://github.com/effl-lab/MaskedKD)
- 옮겨 간 비용:
- 어떻게 쟀나: FLOPs, 처리량(img/s), 분류 정확도(Acc. %), 메모리 사용량 비교.
- 설정: ImageNet-1K, DeiT 학습 레시피. 여러 크기의 학생과 DeiT·CaiT·CLIP·MAE 교사를 사용. 기본 입력 224×224를 16×16 패치 196개로 나눔. DeiT3-H와 MAE-ViT-H는 14×14 패치 256개를 사용.
- 평균 정확도 외 평가: 선형 프로빙 정확도. 효율 지표로 FLOPs·처리량·메모리도 함께 비교.
- 위치:
- 메모: 좋았던 점: 효율 향상을 FLOPs뿐 아니라 처리량과 메모리 등 여러 지표로 확인함.

### 02. DMAE (Masked Autoencoders Enable Efficient Knowledge Distillers) ([arXiv: 2208.12256](https://arxiv.org/abs/2208.12256))
- 효율화 방식: MAE 인코더 중간 층의 특징 출력을 정렬해 증류함. 로짓을 일치시키는 방식은 아님.
- 얻는 것:
- 잃는 것:
  - [구조] 교사의 중간 특징과 패치 위치 대응이 필요함. 교사·학생이 같은 보이는 패치를 사용하는 정렬 방식. [원문](https://arxiv.org/abs/2208.12256)
- 옮겨 간 비용:
- 어떻게 쟀나: Training Cost(GPU-hours). 교사의 사전학습 비용은 별도이며, 증류 학습 비용과 구분해 읽기.
- 설정: ImageNet-1K. MAE 기반 특징 증류 외에 DINO·MoCo-v3 학습에도 적용한 실험이 있음.
- 평균 정확도 외 평가: 4.7절에서 여러 seed의 평균과 표준편차를 제시해 성능 향상의 안정성을 확인함.
- 위치: 4.7절 Standard Deviation Analysis.
- 메모: 좋았던 점: 단일 실행 결과뿐 아니라 표준편차도 제시함. 추가 평가 아이디어(AI 참고): 분류 정확도 외 능력 변화 확인을 위해 잡음·흐림·날씨·디지털 손상에 대한 강건성 평가를 검토. 해당 강건성 손실이 이 논문에서 확인됐다는 뜻은 아님.

### 03. Dyna-DINO ([arXiv: 2606.19483](https://arxiv.org/abs/2606.19483))
- 효율화 방식: 교사의 최종 층을 처음부터 목표로 삼지 않고, 앞쪽 층을 충분히 학습한 뒤 중간 층과 마지막 층으로 증류 목표를 이동함.
- 얻는 것: 학습 FLOPs와 실제 학습시간 감소.
- 잃는 것:
  - [구조] CKA와 목표 층 전환에 교사의 중간 특징이 필요함. 최종 출력만 제공하는 교사에는 그대로 적용할 수 없음. [공식 코드](https://github.com/KevinZ0217/Dyna-DINO)
- 옮겨 간 비용:
- 어떻게 쟀나: FLOPs, 실제 학습시간, 이미지 분류 정확도.
- 설정: 의미 영역 분할은 ADE20K, 인스턴스 검색은 Oxford·Paris 데이터셋에서 평가.
- 평균 정확도 외 평가: 의미 영역 분할: 선형 영역 분할, 인코더 전용 마스크 트랜스포머(EOMT), 다중 스케일(MS) 추론의 세 가지 프로토콜 사용. 인스턴스 검색: 백본에서 이미지별 전역 임베딩을 추출하고, 질의 이미지와 데이터베이스 이미지의 유사도순으로 검색 결과를 정렬해 평균 정밀도(mAP)를 계산.
- 위치: 4.3절의 의미 영역 분할·인스턴스 검색 평가.
- 메모:

### 04. Asymmetric Masked Distillation (AMD) ([arXiv: 2311.03149](https://arxiv.org/abs/2311.03149))
- 효율화 방식: 학생보다 교사의 마스킹 비율을 낮춰 더 많은 문맥을 보게 함. 공통 패치는 직접 정렬하고 교사만 보는 패치는 생성 정렬.
- 얻는 것: 무마스킹 특징 증류 대비 교사 토큰 계산 감소. SSV2 200-epoch 비교에서 무마스킹 32시간, 교사 45%·학생 90% 마스킹 AMD 19시간(Table 7).
- 잃는 것:
  - [구조] 학생이 보는 패치는 교사가 보는 패치의 부분집합이어야 함. 교사·학생 마스크를 서로 독립적으로 정하는 방식은 아님. [§3.3](https://arxiv.org/html/2311.03149v2#S3.SS3)
- 옮겨 간 비용: 특징 생성기·정렬 손실. 대칭 마스킹보다 교사 처리 토큰은 많음.
- 어떻게 쟀나: 실제 학습시간, epoch, 교사·학생 마스킹 비율. 위 시간은 Table 7 안의 비교.
- 설정: SSV2·Kinetics-400 영상, ImageNet-1K 이미지. MAE 기반 교사·학생.
- 평균 정확도 외 평가: AVA 행동 탐지 mAP, UCF101·HMDB51 전이, 행동 종류별 결과, 평균 어텐션 거리.
- 위치: Table 1·7, 부록 Figure 4·5. [원문](https://arxiv.org/html/2311.03149v2)
- 메모:

### 05. MaskAlign ([arXiv: 2211.08887](https://arxiv.org/abs/2211.08887))
- 효율화 방식: 학생이 보는 패치의 특징과 교사가 전체 이미지에서 추출한 대응 특징 사이의 일관성을 학습함. 동적 정렬 모듈(DA)을 사용하며 픽셀 복원 디코더를 제거함.
- 얻는 것: 픽셀 복원 디코더의 계산 제거.
- 잃는 것:
  - [실측·외부] COCO 물체 특징 분류에서 배경을 교체하면 Top-1 77.4→53.8%. 마스킹 자체가 원인인지를 분리한 실험은 아님. [Mind Your Augmentation, Table 3](https://proceedings.iclr.cc/paper_files/paper/2024/file/040321891311e31134a1b3844efb6a9a-Paper-Conference.pdf)
  - [구조] 보이는 패치 위치에 해당하는 교사 특징을 받아야 함. 최종 분류 로짓만 반환하는 교사로는 같은 특징 정렬을 수행할 수 없음. [공식 코드](https://github.com/OpenDriveLab/maskalign)
- 옮겨 간 비용:
- 어떻게 쟀나: 학습 epoch, 시간(Time), 미세조정 정확도(FT Acc. %), 탐지·분할 성능(APbox·APmask).
- 설정: COCO에서 객체 탐지와 인스턴스 분할을 평가.
- 평균 정확도 외 평가: 객체 탐지 APbox, 인스턴스 분할 APmask. 부록의 선형 프로빙·의미 영역 분할·배경 관련 평가도 확인.
- 위치: 4.2절 COCO 평가, 부록 A 선형 프로빙, 부록 B 의미 영역 분할, 부록 C 배경 관련 평가.
- 메모:

### 06. Decoupled KD (DKD) ([arXiv: 2203.08679](https://arxiv.org/abs/2203.08679))
- 효율화 방식: 정답 클래스 지식 증류(TCKD)와 비정답 클래스 지식 증류(NCKD)를 분리하고, 각 손실의 학습 기여도를 따로 조절함. 기존 KD에서 교사의 정답 확신도에 따라 작아지던 NCKD의 계수를 독립적인 가중치로 바꿈.
- 얻는 것:
- 잃는 것:
  - [실측·외부] WRN-40-2→ResNet-20, CIFAR-100-C에서 학생 단독 대비 ECE 0.2258→0.2379(3-seed 평균, 낮을수록 좋음). 통계적 유의성은 확인되지 않음. [CSV](https://github.com/Evowind/confident-dunce/blob/main/figures/seeds_summary.csv)
  - [구조·정의] 이진 분류에서는 정답을 제외하면 클래스가 하나뿐이므로 NCKD의 KL 항은 0이 됨. 다중 비정답 클래스 사이의 관계를 전달할 수 없음. [NCKD 정의](https://arxiv.org/abs/2203.08679)
- 옮겨 간 비용:
- 어떻게 쟀나: 단위 = / teacher forward 셈? / 일회성 비용 셈? / 하드웨어 =
- 설정: 특징 전이 실험에서는 CIFAR-100에서 WRN-40-2 교사로 증류한 WRN-16-2 학생을 특징 추출기로 사용.
- 평균 정확도 외 평가: 특징 추출기를 고정하고 STL-10·Tiny-ImageNet에서 선형 프로빙을 수행해 특징의 전이 가능성을 평가.
- 위치: 4.2절 추가 분석의 특징 전이 실험.
- 메모: NCKD는 정답 클래스를 제외한 클래스들의 예측 분포에 담긴 지식을 다룸. 기존 KD에서는 교사의 정답 확신도가 높을수록 이 부분의 기여가 작아지는 점을 분리해 다룸.

### 07. Information Theoretic Representation Distillation (ITRD) ([arXiv: 2112.00459](https://arxiv.org/abs/2112.00459))
- 효율화 방식: 특징의 상관·상호정보량 기반 손실 사용. 대조 증류의 별도 메모리 뱅크·음성 샘플 저장 없이 계산.
- 얻는 것: 메모리 뱅크 기반 표현 증류 대비 저장·샘플 관리 부담 감소.
- 잃는 것:
  - [구조] 교사·학생의 중간 특징을 배치 단위로 받아 상관·Gram 행렬을 계산함. 로짓만 제공하는 교사 API에는 그대로 적용할 수 없음. [공식 코드](https://github.com/roymiles/ITRD)
- 옮겨 간 비용: 배치 내 상관·Gram 행렬 계산과 학생 특징 투영층.
- 어떻게 쟀나: 추가 연산·메모리 부담. 배치 크기와 특징 차원에 따른 행렬 크기를 구분.
- 설정: CIFAR-100·ImageNet, 여러 CNN 조합. 이진 신경망 증류도 실험.
- 평균 정확도 외 평가: CIFAR-100 특징을 고정한 STL-10·Tiny-ImageNet 선형 프로빙.
- 위치: 손실 정의, 표현 전이 Table 4. [원문](https://arxiv.org/abs/2112.00459)
- 메모:

### 08. Understanding the Role of the Projector in KD ([arXiv: 2303.11098](https://arxiv.org/abs/2303.11098))
- 효율화 방식: 선형 프로젝터·배치 정규화·LogSum 거리로 특징 증류. 별도 메모리 뱅크나 명시적 관계 구조를 만들지 않음.
- 얻는 것: 복잡한 관계 기반 증류 모듈 대신 작은 투영층 사용.
- 잃는 것:
  - [실측·원논문] 정규화 비교에서 프로젝터 가중치의 일부 특이값이 학습 중 0에 가까워지는 현상을 측정함. 특정 인식 능력 손실을 별도로 측정한 결과는 아님. [Figure 2](https://arxiv.org/html/2303.11098v5)
  - [실측·원논문] 큰 MLP 프로젝터에서 입력·출력 특징의 상관이 더 낮아지는 현상을 측정함. [Figure 3](https://arxiv.org/html/2303.11098v5)
- 옮겨 간 비용: 프로젝터 학습·배치 정규화·특징 손실 계산.
- 어떻게 쟀나: 손실 구성의 연산·메모리 규모와 배치 크기·특징 차원.
- 설정: CIFAR-100·ImageNet 분류, CNN→ViT 증류, COCO 탐지.
- 평균 정확도 외 평가: 특이값 변화, 프로젝터 입력·출력 상관, 객체 탐지 AP.
- 위치: Figure 2·3, 다운스트림 평가. [원문](https://arxiv.org/html/2303.11098v5)
- 메모:

### 09. Parameter-Efficient and Student-Friendly Knowledge Distillation (PESF-KD) ([원문](https://arxiv.org/abs/2205.15308))
- 효율화 방식: 고정된 교사 본체 대신 작은 어댑터만 갱신해 교사를 학생에 맞게 적응시킴.
- 얻는 것: 교사 전체를 갱신하는 FT-KD 대비 적은 갱신 파라미터. ImageNet Table 3 batch time: FT-KD 1.44, PESF-KD 0.47, vanilla KD 0.46(표의 단위 그대로).
- 잃는 것:
  - [구조] 교사에 어댑터를 넣고 학습해야 함. 수정할 수 없는 고정 교사 API만으로는 해당 교사 적응을 수행할 수 없음. [저자 코드](https://github.com/1170300319/PESF-KD)
- 옮겨 간 비용: 어댑터 학습·교사 순전파. frozen-teacher KD에 없던 교사 적응 비용이 추가됨.
- 어떻게 쟀나: 갱신 파라미터 수, batch time. FT-KD 대비 절감과 vanilla KD 대비 비용을 구분.
- 설정: CIFAR-100·ImageNet. ImageNet ResNet50→ResNet18. NLP GLUE도 포함.
- 평균 정확도 외 평가: 교사·학생 KL divergence, 예측 일치율(Agree), 논문 정의의 PL·Gap.
- 위치: Figure 1, Table 3, 일관성 분석. [저자 코드](https://github.com/1170300319/PESF-KD)

### 10. Knowledge Distillation from A Stronger Teacher (DIST) ([원문](https://arxiv.org/abs/2205.10536))
- 효율화 방식: 예측 확률의 정확한 일치 대신 클래스 간·샘플 간 Pearson 상관을 정렬.
- 얻는 것: 특징 정렬 모듈·메모리 뱅크 없이 로짓에서 계산. 논문은 vanilla KD와 거의 같은 학습 비용이라고 설명.
- 잃는 것:
  - [실측·외부] CLIP ViT-L/14→ViT-B/16에서 일반 KD 대비 ImageNet-R 51.5→40.8%, Sketch 34.7→26.9%. [외부 Table 3](https://arxiv.org/html/2311.01441v2#S4)
  - [실측·외부] 같은 실험에서 ImageNet-C mCE 61.5→65.8(낮을수록 좋음), Stylized-ImageNet 정확도 16.2→12.7%. [같은 표](https://arxiv.org/html/2311.01441v2#S4)
- 옮겨 간 비용: 클래스·배치 방향의 상관 계산. 강한 교사를 쓰는 경우 교사 순전파 비용도 별도 고려.
- 어떻게 쟀나: 원논문의 비용 설명과 다운스트림 결과. vanilla KD 대비 정량 가속률을 임의로 부여하지 않음.
- 설정: CIFAR-100·ImageNet, 강한 교사와 다양한 학생. 객체 탐지·분할에도 적용.
- 평균 정확도 외 평가: 탐지 AP, Cityscapes mIoU. 외부 연구는 ImageNet-C·R·Sketch·A·Stylized-ImageNet 평가.
- 위치: §1·3, 탐지·분할 평가. 외부 강건성 Table 3. [공식 코드](https://github.com/hunto/DIST_KD)
