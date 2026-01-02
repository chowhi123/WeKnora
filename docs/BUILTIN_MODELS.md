# 내장 모델 관리 가이드

## 개요

내장 모델은 시스템 수준의 모델 구성으로, 모든 테넌트에게 표시되지만 민감한 정보는 숨겨지며 편집하거나 삭제할 수 없습니다. 내장 모델은 일반적으로 시스템 기본 모델 구성을 제공하는 데 사용되어 모든 테넌트가 통일된 모델 서비스를 사용할 수 있도록 합니다.

## 내장 모델 특성

- **모든 테넌트에게 표시**: 내장 모델은 별도 구성 없이 모든 테넌트에게 표시됩니다.
- **보안 보호**: 내장 모델의 민감한 정보(API Key, Base URL)는 숨겨져 세부 정보를 볼 수 없습니다.
- **읽기 전용 보호**: 내장 모델은 편집하거나 삭제할 수 없으며 기본 모델로만 설정할 수 있습니다.
- **통합 관리**: 시스템 관리자가 통합 관리하여 구성의 일관성과 보안을 보장합니다.

## 내장 모델 추가 방법

내장 모델은 데이터베이스를 통해 직접 삽입해야 합니다. 다음은 내장 모델을 추가하는 단계입니다.

### 1. 모델 데이터 준비

먼저, 내장 모델로 설정할 모델 구성 정보가 있는지 확인하십시오. 여기에는 다음이 포함됩니다.
- 모델 이름 (name)
- 모델 유형 (type): `KnowledgeQA`, `Embedding`, `Rerank` 또는 `VLLM`
- 모델 소스 (source): `local` 또는 `remote`
- 모델 매개변수 (parameters): base_url, api_key, provider 등 포함
- 테넌트 ID (tenant_id): 충돌을 피하기 위해 10000 미만의 테넌트 ID를 사용하는 것이 좋습니다.

**지원되는 제공자 (provider)**: `generic` (사용자 정의), `openai`, `aliyun`, `zhipu`, `volcengine`, `hunyuan`, `deepseek`, `minimax`, `mimo`, `siliconflow`, `jina`, `openrouter`, `gemini`

### 2. SQL 삽입 문 실행

다음 SQL 문을 사용하여 내장 모델을 삽입합니다.

```sql
-- 예시: LLM 내장 모델 삽입
INSERT INTO models (
    id,
    tenant_id,
    name,
    type,
    source,
    description,
    parameters,
    is_default,
    status,
    is_builtin
) VALUES (
    'builtin-llm-001',                    -- 고정 ID 사용, builtin- 접두사 사용 권장
    10000,                                -- 테넌트 ID (첫 번째 테넌트 사용)
    'GPT-4',                              -- 모델 이름
    'KnowledgeQA',                        -- 모델 유형
    'remote',                             -- 모델 소스
    '내장 LLM 모델',                       -- 설명
    '{"base_url": "https://api.openai.com/v1", "api_key": "sk-xxx", "provider": "openai"}'::jsonb,  -- 매개변수 (JSON 형식)
    false,                                -- 기본값 여부
    'active',                             -- 상태
    true                                  -- 내장 모델로 표시
) ON CONFLICT (id) DO NOTHING;

-- 예시: Embedding 내장 모델 삽입
INSERT INTO models (
    id,
    tenant_id,
    name,
    type,
    source,
    description,
    parameters,
    is_default,
    status,
    is_builtin
) VALUES (
    'builtin-embedding-001',
    10000,
    'text-embedding-ada-002',
    'Embedding',
    'remote',
    '내장 Embedding 모델',
    '{"base_url": "https://api.openai.com/v1", "api_key": "sk-xxx", "provider": "openai", "embedding_parameters": {"dimension": 1536, "truncate_prompt_tokens": 0}}'::jsonb,
    false,
    'active',
    true
) ON CONFLICT (id) DO NOTHING;

-- 예시: ReRank 내장 모델 삽입
INSERT INTO models (
    id,
    tenant_id,
    name,
    type,
    source,
    description,
    parameters,
    is_default,
    status,
    is_builtin
) VALUES (
    'builtin-rerank-001',
    10000,
    'bge-reranker-base',
    'Rerank',
    'remote',
    '내장 ReRank 모델',
    '{"base_url": "https://api.jina.ai/v1", "api_key": "jina-xxx", "provider": "jina"}'::jsonb,
    false,
    'active',
    true
) ON CONFLICT (id) DO NOTHING;

-- 예시: VLLM 내장 모델 삽입
INSERT INTO models (
    id,
    tenant_id,
    name,
    type,
    source,
    description,
    parameters,
    is_default,
    status,
    is_builtin
) VALUES (
    'builtin-vllm-001',
    10000,
    'gpt-4-vision',
    'VLLM',
    'remote',
    '내장 VLLM 모델',
    '{"base_url": "https://dashscope.aliyuncs.com/compatible-mode/v1", "api_key": "sk-xxx", "provider": "aliyun"}'::jsonb,
    false,
    'active',
    true
) ON CONFLICT (id) DO NOTHING;
```

### 3. 삽입 결과 확인

다음 SQL 쿼리를 실행하여 내장 모델이 성공적으로 삽입되었는지 확인합니다.

```sql
SELECT id, name, type, is_builtin, status 
FROM models 
WHERE is_builtin = true
ORDER BY type, created_at;
```

## 주의 사항

1. **ID 명명 규칙**: `builtin-{type}-{번호}` 형식을 사용하는 것이 좋습니다. 예: `builtin-llm-001`, `builtin-embedding-001`
2. **테넌트 ID**: 내장 모델은 모든 테넌트에 속할 수 있지만 첫 번째 테넌트 ID(일반적으로 10000)를 사용하는 것이 좋습니다.
3. **매개변수 형식**: `parameters` 필드는 유효한 JSON 형식이어야 합니다.
4. **멱등성**: `ON CONFLICT (id) DO NOTHING`을 사용하여 반복 실행 시 오류가 발생하지 않도록 합니다.
5. **보안**: 내장 모델의 API Key와 Base URL은 프론트엔드에서 자동으로 숨겨지지만 데이터베이스의 원본 데이터는 여전히 존재하므로 데이터베이스 액세스 권한을 잘 관리하십시오.

## 기존 모델을 내장 모델로 설정

이미 모델이 있고 이를 내장 모델로 설정하려면 UPDATE 문을 사용할 수 있습니다.

```sql
UPDATE models 
SET is_builtin = true 
WHERE id = '모델ID' AND name = '모델이름';
```

## 내장 모델 제거

내장 모델 표시를 제거(일반 모델로 복원)하려면 다음을 실행합니다.

```sql
UPDATE models 
SET is_builtin = false 
WHERE id = '모델ID';
```

주의: 내장 모델 표시를 제거하면 해당 모델은 일반 모델로 복원되어 편집 및 삭제가 가능해집니다.
