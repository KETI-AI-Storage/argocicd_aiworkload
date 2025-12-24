# KETI AI Storage - Argo CD Workload Test

KETI AI Storage 시스템의 Argo CD 연동 테스트용 워크로드 레포지토리입니다.

## 워크로드 구성

### TEXT Workloads (NLP)
| 워크로드 | 타입 | 프레임워크 | 설명 |
|---------|------|-----------|------|
| `llama-training` | Job | PyTorch + HuggingFace | LLaMA 모델 학습 (약 25분) |
| `bert-inference` | Deployment | PyTorch + HuggingFace | BERT 추론 서비스 (지속 실행) |

### IMAGE Workloads (Computer Vision)
| 워크로드 | 타입 | 프레임워크 | 설명 |
|---------|------|-----------|------|
| `resnet-training` | Job | PyTorch + TorchVision | ResNet50 이미지 분류 학습 (약 20분) |
| `yolo-inference` | Deployment | PyTorch + Ultralytics | YOLOv8 객체 탐지 서비스 (지속 실행) |

## 주요 특징

- **insight-trace 사이드카**: 모든 워크로드에 insight-trace 사이드카가 포함되어 동적 분석 수행
- **shareProcessNamespace**: 사이드카가 메인 컨테이너의 프로세스를 감지할 수 있도록 설정
- **APOLLO 연동**: insight-trace가 APOLLO로 WorkloadSignature 전송
- **장시간 실행**: 타입 감지를 위해 충분히 오래 실행되도록 설계

## 사용 방법

### 1. Argo CD Application 등록
```bash
kubectl apply -f argocd/application.yaml
```

### 2. 동기화 상태 확인
```bash
# CLI
argocd app get keti-ai-workload-test

# 또는 UI
kubectl port-forward svc/argocd-server -n argocd 8443:443
# https://localhost:8443 접속
```

### 3. 워크로드 상태 확인
```bash
kubectl get pods -n ai-workload-test
```

### 4. 모니터링
```bash
# 통합 모니터링 (insight-scope + insight-trace + APOLLO)
/root/workspace/integration-test/scripts/monitor-all.sh ai-workload-test
```

## 디렉토리 구조

```
argocicd_aiworload/
├── README.md
├── argocd/
│   └── application.yaml      # Argo CD Application 정의
└── workloads/
    ├── kustomization.yaml    # Kustomize 설정
    ├── namespace.yaml        # ai-workload-test 네임스페이스
    ├── text/
    │   ├── llama-training.yaml
    │   └── bert-inference.yaml
    └── image/
        ├── resnet-training.yaml
        └── yolo-inference.yaml
```

## 프로세스 키워드 (insight-trace 감지용)

| 워크로드 | 프로세스 이름 | 감지 키워드 |
|---------|--------------|------------|
| llama-training | `llama_pytorch_huggingface_trainer.py` | llama, pytorch, huggingface |
| bert-inference | `bert_transformer_pytorch_inference.py` | bert, transformer, pytorch |
| resnet-training | `resnet50_torchvision_image_trainer.py` | resnet, torchvision, image |
| yolo-inference | `yolov8_opencv_image_detector.py` | yolo, opencv, image, detector |

## 관련 프로젝트

- [insight-trace](https://github.com/KETI-AI-Storage/insight-trace): 런타임 워크로드 분석 사이드카
- [insight-scope](https://github.com/KETI-AI-Storage/insight-scope): 정적 YAML 분석
- [apollo](https://github.com/KETI-AI-Storage/apollo): 스토리지 정책 서버
