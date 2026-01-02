# WeKnora 지식 그래프

## 빠른 시작

- .env 관련 환경 변수 구성
    - Neo4j 활성화: `NEO4J_ENABLE=true`
    - Neo4j URI: `NEO4J_URI=bolt://neo4j:7687`
    - Neo4j 사용자 이름: `NEO4J_USERNAME=neo4j`
    - Neo4j 비밀번호: `NEO4J_PASSWORD=password`

- Neo4j 시작
```bash
docker-compose --profile neo4j up -d
```

- 지식 베이스 설정 페이지에서 엔티티 및 관계 추출을 활성화하고 지침에 따라 관련 내용을 구성합니다.

## 그래프 생성

문서를 업로드하면 시스템이 자동으로 엔티티와 관계를 추출하고 해당 지식 그래프를 생성합니다.

![지식 그래프 예시](./images/graph3.png)

## 그래프 보기

`http://localhost:7474`에 로그인하고 `match (n) return (n)`을 실행하여 생성된 지식 그래프를 확인합니다.

대화 시 시스템은 자동으로 지식 그래프를 조회하고 관련 지식을 가져옵니다.
