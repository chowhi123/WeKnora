# 자주 묻는 질문 (FAQ)

## 1. 로그는 어떻게 확인하나요?
```bash
docker compose logs -f app docreader postgres
```

## 2. 서비스를 시작하고 중지하려면 어떻게 해야 하나요?
```bash
# 서비스 시작
./scripts/start_all.sh

# 서비스 중지
./scripts/start_all.sh --stop

# 데이터베이스 정리
./scripts/start_all.sh --stop && make clean-db
```

## 3. 서비스 시작 후 문서를 정상적으로 업로드할 수 없나요?

일반적으로 Embedding 모델과 대화 모델이 올바르게 설정되지 않아 발생합니다. 다음 단계에 따라 문제를 해결하세요.

1. `.env` 구성의 모델 정보가 완전하게 구성되었는지 확인하세요. ollama를 사용하여 로컬 모델에 액세스하는 경우 로컬 ollama 서비스가 정상적으로 실행 중인지 확인해야 하며, 동시에 `.env`의 다음 환경 변수가 올바르게 설정되어야 합니다.
```bash
# LLM 모델
INIT_LLM_MODEL_NAME=your_llm_model
# Embedding 모델
INIT_EMBEDDING_MODEL_NAME=your_embedding_model
# Embedding 모델 벡터 차원
INIT_EMBEDDING_MODEL_DIMENSION=your_embedding_model_dimension
# Embedding 모델 ID, 일반적으로 문자열
INIT_EMBEDDING_MODEL_ID=your_embedding_model_id
```

remote api를 통해 모델에 액세스하는 경우 해당 `BASE_URL` 및 `API_KEY`를 추가로 제공해야 합니다.
```bash
# LLM 모델 액세스 주소
INIT_LLM_MODEL_BASE_URL=your_llm_model_base_url
# LLM 모델 API 키, 인증이 필요한 경우 설정
INIT_LLM_MODEL_API_KEY=your_llm_model_api_key
# Embedding 모델 액세스 주소
INIT_EMBEDDING_MODEL_BASE_URL=your_embedding_model_base_url
# Embedding 모델 API 키, 인증이 필요한 경우 설정
INIT_EMBEDDING_MODEL_API_KEY=your_embedding_model_api_key
```

재정렬(Rerank) 기능이 필요한 경우 Rerank 모델을 추가로 구성해야 하며, 구체적인 구성은 다음과 같습니다.
```bash
# 사용하는 Rerank 모델 이름
INIT_RERANK_MODEL_NAME=your_rerank_model_name
# Rerank 모델 액세스 주소
INIT_RERANK_MODEL_BASE_URL=your_rerank_model_base_url
# Rerank 모델 API 키, 인증이 필요한 경우 설정
INIT_RERANK_MODEL_API_KEY=your_rerank_model_api_key
```

2. 메인 서비스 로그를 확인하여 `ERROR` 로그 출력이 있는지 확인하세요.

## 4. 멀티모달 기능을 활성화하려면 어떻게 해야 하나요?
1. `.env`의 다음 구성이 올바르게 설정되었는지 확인하세요.
```bash
# VLM_MODEL_NAME 사용하는 멀티모달 모델 이름
VLM_MODEL_NAME=your_vlm_model_name

# VLM_MODEL_BASE_URL 사용하는 멀티모달 모델 액세스 주소
VLM_MODEL_BASE_URL=your_vlm_model_base_url

# VLM_MODEL_API_KEY 사용하는 멀티모달 모델 API 키
VLM_MODEL_API_KEY=your_vlm_model_api_key
```
참고: 멀티모달 대규모 모델은 현재 remote api 액세스만 지원하므로 `VLM_MODEL_BASE_URL` 및 `VLM_MODEL_API_KEY`를 제공해야 합니다.

2. 파싱된 파일을 COS(Cloud Object Storage)에 업로드해야 하므로 `.env`의 `COS` 정보가 올바르게 설정되었는지 확인하세요.
```bash
# Tencent Cloud COS Secret ID
COS_SECRET_ID=your_cos_secret_id

# Tencent Cloud COS Secret Key
COS_SECRET_KEY=your_cos_secret_key

# Tencent Cloud COS 지역, 예: ap-guangzhou
COS_REGION=your_cos_region

# Tencent Cloud COS 버킷 이름
COS_BUCKET_NAME=your_cos_bucket_name

# Tencent Cloud COS 앱 ID
COS_APP_ID=your_cos_app_id

# 파일 저장을 위한 Tencent Cloud COS 경로 접두사
COS_PATH_PREFIX=your_cos_path_prefix
```
중요: COS 내 파일 권한을 반드시 **공개 읽기(Public Read)** 로 설정해야 합니다. 그렇지 않으면 문서 파싱 모듈이 파일을 정상적으로 파싱할 수 없습니다.

3. 문서 파싱 모듈 로그를 확인하여 OCR 및 캡션이 올바르게 파싱되고 출력되는지 확인하세요.


## 5. 데이터 분석 기능은 어떻게 사용하나요?

데이터 분석 기능을 사용하기 전에 에이전트가 관련 도구로 구성되어 있는지 확인하세요.

1. **지능형 추론**: 도구 구성에서 다음 두 가지 도구를 선택해야 합니다.
   - 데이터 메타 정보 보기
   - 데이터 분석

2. **빠른 질의응답 에이전트**: 도구를 수동으로 선택할 필요 없이 간단한 데이터 조회 작업을 직접 수행할 수 있습니다.

### 주의 사항 및 사용 지침

1. **지원되는 파일 형식**
   - 현재 **CSV** (`.csv`) 및 **Excel** (`.xlsx`, `.xls`) 형식의 파일만 지원합니다.
   - 복잡한 Excel 파일의 경우 읽기에 실패하면 표준 CSV 형식으로 변환한 후 다시 업로드하는 것이 좋습니다.

2. **쿼리 제한**
   - **읽기 전용 쿼리**만 지원하며, `SELECT`, `SHOW`, `DESCRIBE`, `EXPLAIN`, `PRAGMA` 등의 문이 포함됩니다.
   - 데이터 수정 작업(예: `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP` 등)의 실행은 금지됩니다.


## P.S.
위의 방법으로 문제가 해결되지 않으면 issue에 문제를 설명하고 문제 해결을 돕기 위해 필요한 로그 정보를 제공해 주세요.
