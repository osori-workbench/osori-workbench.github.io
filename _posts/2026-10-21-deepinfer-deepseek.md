---
title: "DeepInfer 공개: 'DeepSeek 프로덕션 엔진'이라는데, 서빙 스택의 절반만 들어 있다"
date: 2026-10-21 22:37:10 +0900
categories: [Dev, Daily]
tags: [llm, inference, deepseek, gpu]
image:
  path: /assets/img/posts/2026-10-21-deepinfer-deepseek.png
  alt: "DeepInfer 공개: 'DeepSeek 프로덕션 엔진'이라는데, 서빙 스택의 절반만 들어 있다"
---


- DeepSeek이 자사 대규모 프로덕션 서비스를 돌린다는 추론 프레임워크 DeepInfer를 Apache 2.0으로 공개했다. 지원 모델은 DeepSeek-V4.1-Flash 하나다.
- 비동기 스케줄링, 배치 불변 추론, 결정적 샘플링, 구조화 출력 등 기능 목록은 화려하지만, KV 캐시 시스템과 로드 밸런싱은 이번 릴리스에서 빠져 있다.
- NVIDIA SM100과 Huawei Ascend 950을 둘 다 지원한다. 다만 몇몇 기능은 NVIDIA 전용이다.

## 도대체 이게 왜 떴는가

이유는 세 가지로 보인다. 첫째, 만든 곳이 DeepSeek이다. 스타 16.9k, 포크 1.7k인데 커밋은 4개다. 코드 품질보다 출처가 만든 숫자다. 커밋 히스토리가 얇다는 건 내부 저장소를 스냅샷으로 떠서 올렸다는 뜻일 가능성이 높다. 다만 자료만으로는 확정할 수 없다.

둘째, Ascend 950 NPU 지원이다. HN 댓글에서도 "엔비디아 독점에 균열이 생길 수 있다"는 반응이 나왔다. 실제로 그럴지는 모르겠다. README의 요구사항이 CANN 9.2에 전용 requirements 파일까지 따로 있는 걸 보면 시늉 수준은 아니지만, 외부 팀이 Ascend 장비를 구해서 돌릴 수 있느냐는 별개 문제다.

셋째, 댓글에 인용된 숫자다. 단일 클러스터 하루 155조 토큰, 카드당 시간당 매출 38달러라는 출처는 지후(Zhihu) 글이다. 이 레포 README에는 없다. 나는 검증하지 못했으니 "그렇다더라" 수준으로만 받아야 한다. 정치적 타이밍 음모론 댓글도 있는데, 근거 없는 추측이라 다루지 않는다.

## 핵심 동작 원리

README가 밝힌 것만 정리한다. 내부 구현까지는 자료에 없으니 추측하지 않는다.

- 커널: FlashMLA, DeepGEMM, DeepEP, TileKernels, DeepSelect, 그리고 커스텀 TileLang 커널을 쓴다.
- 병렬화: Attention DP / balanced CP, MoE는 EP + EPLB(전문가 병렬과 로드 밸런싱) 조합이다.
- 비동기 스케줄링: CPU의 배치 스케줄링과 디바이스 실행을 겹친다. 기본 활성화다.
- DSpark와 신뢰도 기반 검증: 드래프트 토큰으로 디코딩을 가속하고, 신뢰도 추정으로 검증 시점을 스케줄링한다. DEEPINFER_ENABLE_MTP_VARLEN=1로 켜고, DEEPINFER_USE_NEXTN_DECODE가 최대 드래프트 길이(기본 5)를 정한다.
- SWA bounded replay: 디코더 프리필 작업을 줄이려고 제한된 길이의 접미부만 재생한다. V4.1 기본값은 DEEPINFER_YOCO_PREFILL_TRUNCATION_MODE=compact, DEEPINFER_YOCO_PREFILL_KEEP_TOKENS=140이다.

개인적으로 제일 실무적인 건 재현성 쪽이다. DEEPINFER_BATCH_INVARIANT=1이면 배치 구성이 달라도 결과가 같다. 결정적 샘플링은 거기에 DEEPINFER_ENABLE_MTP_VARLEN=0을 더하고 SamplingParams에 고정 seq_seed를 넘기는 방식이다. 둘 다 NVIDIA 전용이다. 장애 재현이나 회귀 테스트에서 "같은 입력인데 왜 출력이 다르죠" 싸움을 끝내줄 수 있다. 성능 비용이 얼마인지는 자료에 없다.

## 다른 선택지와 비교

README에 따르면 DeepInfer는 원래 vLLM 위에 주로 만들어졌고, SGLang과 TensorRT-LLM에서 영감을 받았다. 그러니 "vLLM 포크에서 출발해 DeepSeek 모델에 극단적으로 맞춘 물건"으로 읽는 게 안전하다.

범용성은 vLLM, SGLang이 훨씬 낫다. 모델 하나(V4.1-Flash)만 지원하는 프레임워크와 비교할 수준이 아니다. 반대로 이 모델을 쓴다면 모델 구조에 맞춘 커널과 기능이 이미 들어 있다는 게 장점이다. 처리량이나 지연시간을 vLLM 등과 직접 비교한 수치는 자료에 없다. README에 벤치마크 항목은 있지만(프리필 4장, 디코드 64장의 SM100 GPU 기준), 구체적 숫자는 내가 확인하지 못했다.

## 현업 적용 시 함정

여기서 삽질하기 딱 좋다.

1. 오프라인 추론만 된다. README가 "완전한 추론 서비스에 필요하지만 이번에 포함되지 않은 것"으로 KV 캐시 시스템, VE/Prefill/Decode 분리, 프리필·디코드 DP 레플리카 간 로드 밸런싱을 명시했다. 즉 API 서버를 붙여 바로 서비스한다는 기대는 접어야 한다. 이 셋을 직접 만들거나 다른 걸로 대체해야 한다.
2. 하드웨어 장벽이 높다. NVIDIA SM100(Blackwell 계열 표기로 보이지만 자료에는 명시가 없다)과 Ascend 950만 된다. 이전 세대 GPU로는 시도조차 어렵다고 보는 게 맞다.
3. 빌드 환경이 까다롭다. Linux, GCC 13 이상, Python 3.12, CUDA 13(또는 CANN 9.2)이 필요하다. 빈 가상환경에서 build.sh를 돌려야 하고, DEEPINFER_ACCELERATOR 환경변수를 실행 환경에도 유지해야 한다. 이거 빼먹으면 이상한 에러를 만난다.
4. 환경변수 기능이 많다. 기능 플래그 상호작용(예: 결정적 샘플링은 MTP_VARLEN을 꺼야 함)이 문서 한 줄에 묻혀 있다. 값을 잘못 조합하면 조용히 비결정적이 될 가능성이 있다.
5. 성숙도를 알 수 없다. 커밋 4개라 이슈 대응이나 릴리스 규칙이 어떨지는 아직 모른다.

## 한줄평

엔진 본체는 공개됐고 서빙에 필요한 바퀴 몇 개는 빠진 상태다. 연구용이나 구조 학습용으로는 보물이지만, 당장 프로덕션에 올리려면 빠진 부분부터 직접 만들 각오를 해야 한다.

---

*원문: [DeepInfer: High-performance inference framework for DeepSeek models](https://github.com/deepseek-ai/DeepInfer) (via hn)*
