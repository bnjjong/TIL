[github link](https://github.com/getzep/graphiti)

# Graphiti: 시간 인식 지식 그래프를 통한 AI 에이전트 메모리 혁신

Graphiti는 Zep AI에서 개발한 오픈소스 프레임워크로, AI 에이전트를 위한 동적이고 시간 인식이 가능한 지식 그래프 구축을 목적으로 합니다. 전통적인 검색 증강 생성(RAG) 시스템의 정적 문서 검색 한계를 극복하고, 실시간으로 변화하는 데이터를 효과적으로 처리할 수 있는 혁신적인 접근 방식을 제공합니다. 이 도구는 단순한 지식 그래프 생성을 넘어서, 시간의 흐름에 따른 관계 변화를 추적하고 역사적 맥락을 유지하면서도 실시간 업데이트를 지원하는 포괄적인 메모리 레이어 서비스입니다.[^1][^2][^3]

## 핵심 아키텍처와 기술적 특징

### 시간 인식 지식 그래프 구조

Graphiti의 핵심은 G = (N, E, φ)로 표현되는 삼층 구조의 시간 인식 지식 그래프입니다. 이 구조는 **에피소드 서브그래프(Episode Subgraph)**, **의미적 엔티티 서브그래프(Semantic Entity Subgraph)**, **커뮤니티 서브그래프(Community Subgraph)**로 구성됩니다. 에피소드 서브그래프는 원시 입력 데이터인 메시지, 텍스트, JSON 형식의 데이터를 손실 없이 저장하는 역할을 담당하며, 의미적 엔티티와 관계 추출의 기반이 됩니다.[^3][^4]

의미적 엔티티 서브그래프는 에피소드에서 추출된 엔티티들과 그들 간의 관계를 나타내며, 각 엣지(edge)에는 유효 기간(t_valid, t_invalid)을 포함한 시간적 메타데이터가 포함됩니다. 이러한 양시간적(bi-temporal) 모델은 이벤트 발생 시간과 데이터 수집 시간을 명시적으로 추적하여, 정확한 시점 쿼리와 역사적 분석을 가능하게 합니다.[^1][^2][^5]

### 실시간 증분 업데이트 시스템

Graphiti의 가장 중요한 특징 중 하나는 **실시간 증분 업데이트** 능력입니다. 새로운 데이터 에피소드가 들어올 때마다 전체 그래프를 재계산하지 않고, 기존 엔티티와 관계에 대해 즉시 해결하고 업데이트합니다. 이 과정에서 의미적, 키워드, 그래프 검색을 통해 새로운 지식이 기존 지식과 충돌하는지 판단하고, 충돌이 발생할 경우 시간적 메타데이터를 활용하여 구식 정보를 지능적으로 무효화하되 삭제하지는 않습니다.[^1][^2][^6]

![Traditional RAG vs GraphRAG vs Graphiti 비교표](https://ppl-ai-code-interpreter-files.s3.amazonaws.com/web/direct-files/a1d8b49704a854a1f5c1213b0c1fc2c5/6377ba4b-8597-4276-86af-d11b4c581beb/511a2c7c.png)

Traditional RAG vs GraphRAG vs Graphiti 비교표

### 하이브리드 검색 및 검색 시스템

Graphiti는 **의미적 임베딩**, **키워드(BM25) 검색**, **직접 그래프 순회**를 결합한 하이브리드 검색 방식을 채택합니다. 이 접근 방식은 검색 시 LLM 호출을 전혀 필요로 하지 않아 매우 낮은 지연 시간을 달성합니다. Neo4j의 벡터 및 BM25 인덱스 지원을 활용하여 그래프 크기에 관계없이 거의 일정한 시간 내에 노드와 엣지에 접근할 수 있습니다.[^1][^2][^7]

## 기존 접근법과의 차별화

### 전통적 RAG 시스템 대비 장점

전통적인 RAG 시스템이 정적 문서 청크에 의존하고 전체 재인덱싱이 필요한 반면, Graphiti는 연속적이고 점진적인 업데이트를 지원합니다. 또한 평면적인 문서 임베딩 대신 에피소드 데이터, 의미적 엔티티, 커뮤니티로 구성된 계층적 지식 구조를 활용합니다.[^1][^5]

### Microsoft GraphRAG와의 비교

Microsoft의 GraphRAG는 정적 문서 요약에 특화되어 있고 배치 지향 처리 방식을 사용하는 반면, Graphiti는 동적 데이터 관리에 최적화되어 있습니다. GraphRAG가 순차적 LLM 요약화에 의존하여 몇 초에서 수십 초의 쿼리 지연 시간을 보이는 반면, Graphiti는 일반적으로 1초 이하의 지연 시간을 달성합니다. 특히 충돌 처리 방식에서 GraphRAG가 LLM 기반 요약화 판단에 의존하는 반면, Graphiti는 시간적 엣지 무효화 방식을 사용하여 더 정확하고 효율적인 처리를 제공합니다.[^1][^5]

![Graphiti 성능 벤치마크 결과](https://ppl-ai-code-interpreter-files.s3.amazonaws.com/web/direct-files/a1d8b49704a854a1f5c1213b0c1fc2c5/936e8e29-6756-4ae4-be18-2a93ff131b78/b43d2037.png)

Graphiti 성능 벤치마크 결과

## 성능 벤치마크 및 실증 결과

### Deep Memory Retrieval (DMR) 벤치마크

Graphiti는 MemGPT 팀이 설정한 주요 평가 지표인 DMR 벤치마크에서 현재 최고 수준의 메모리 시스템인 MemGPT를 능가하는 성능을 보였습니다. 구체적으로 Graphiti는 94.8%의 정확도를 기록하여 MemGPT의 93.4%를 상회했습니다.[^1][^3][^8]

### LongMemEval 벤치마크에서의 뛰어난 성과

더 중요한 것은 실제 기업 사용 사례를 더 잘 반영하는 LongMemEval 벤치마크에서 Graphiti가 보인 탁월한 성능입니다. 이 벤치마크에서 Graphiti는 **최대 18.5%의 정확도 향상**을 달성하면서 동시에 **응답 지연 시간을 90% 단축**시켰습니다. 특히 크로스 세션 정보 합성과 장기 컨텍스트 유지와 같은 기업 핵심 작업에서 성능 향상이 두드러졌습니다.[^1][^3][^8]

### 실시간 성능 지표

Zep의 Graphiti 구현은 P95 지연 시간 300ms의 매우 낮은 지연 검색을 달성합니다. 이는 음성 애플리케이션을 포함한 실시간 상호작용에 이상적인 수준입니다. 또한 기본 구성에서 10개의 동시 작업으로 설정되어 있지만, LLM 제공업체가 더 높은 처리량을 허용하는 경우 동시성을 증가시켜 에피소드 수집 성능을 향상시킬 수 있습니다.[^2][^9]

![A colorful network graph visualization showing interconnected nodes representing a knowledge graph with distinct clusters and relationships.](https://pplx-res.cloudinary.com/image/upload/v1754722127/pplx_project_search_images/3f5f1c1b3428cac4e1a91c4a103846113074e58f.png)

A colorful network graph visualization showing interconnected nodes representing a knowledge graph with distinct clusters and relationships.

## 실제 적용 사례 및 활용 분야

### 엔터프라이즈 애플리케이션

Graphiti는 **CRM 시스템**, **청구 플랫폼**, **고객 지원 시스템** 등 다양한 비즈니스 시스템의 동적 데이터와 개인 지식을 융합하는 어시스턴트 구축에 활용됩니다. 사용자 상호작용으로부터 학습하면서 비즈니스 시스템의 실시간 데이터를 통합하여 개인화된 경험을 제공할 수 있습니다.[^1][^5]

### AI 에이전트의 상태 기반 추론

복잡한 작업을 자율적으로 수행하는 에이전트의 경우, 교통 상황이나 실시간 음성 전사와 같은 다양한 동적 소스로부터 상태 변화를 처리하며 추론할 수 있습니다. 이는 Graphiti의 시간적 추론 능력과 동적 데이터 처리 기능이 결합된 결과입니다.[^4][^5][^6]

### 맞춤형 엔티티 정의를 통한 도메인 특화 애플리케이션

Graphiti는 **Pydantic 모델**을 사용하여 도메인별 엔티티 타입을 정의할 수 있는 직관적인 방법을 제공합니다. 예를 들어 고객의 개인화된 선호도와 관심사(좋아하는 레스토랑, 연락처, 취미), 표준 속성(이름, 생년월일, 주소), 절차적 메모리(작업 지시사항), 도메인별 비즈니스 객체(제품, 판매 주문) 등을 정의할 수 있습니다.[^2]

![Graphster knowledge graph architecture showing data ingestion, semantic graph construction using NLP, AI-based pattern analysis, and insight generation modules.](https://pplx-res.cloudinary.com/image/upload/v1756007659/pplx_project_search_images/a2cb18970ce64682455087b1d219b0df5b954b52.png)

Graphster knowledge graph architecture showing data ingestion, semantic graph construction using NLP, AI-based pattern analysis, and insight generation modules.

## 기술적 구현 및 설치 환경

### 시스템 요구사항 및 호환성

Graphiti는 **Python 3.10 이상**을 요구하며, 그래프 데이터베이스로 **Neo4j 5.26**, **FalkorDB 1.1.2**, 또는 **Amazon Neptune Database Cluster**를 지원합니다. OpenAI API 키가 기본 설정으로 필요하지만, Google Gemini, Anthropic, Groq 등 다른 LLM 제공업체도 선택적으로 지원됩니다.[^9]

### 다양한 LLM 제공업체 지원

Graphiti는 **구조화된 출력**을 지원하는 LLM 서비스와 최적으로 작동하며, OpenAI와 Gemini를 주로 권장합니다. Azure OpenAI의 경우 별도의 엔드포인트 구성을 통해 LLM 추론과 임베딩 서비스를 분리하여 사용할 수 있으며, Ollama를 통한 로컬 LLM 실행도 지원합니다.[^9]

### 데이터베이스 백엔드 유연성

Neo4j의 경우 기본적으로 `neo4j` 데이터베이스를 사용하지만, 사용자 정의 데이터베이스명을 지정할 수 있습니다. FalkorDB는 Docker를 통한 즉시 시작이 가능하며 `default_db`를 기본 데이터베이스로 사용합니다. Amazon Neptune의 경우 Neptune Database와 OpenSearch Serverless 컬렉션을 전문 검색 백엔드로 함께 사용합니다.[^9]

![A knowledge graph visualizing Boeing aircraft models, their manufacturers, related airlines, and countries with interactive link options.](https://pplx-res.cloudinary.com/image/upload/v1755010239/pplx_project_search_images/1ecb65297485920f3e6484e6507e42ca1176275a.png)

A knowledge graph visualizing Boeing aircraft models, their manufacturers, related airlines, and countries with interactive link options.

## 커뮤니티 생태계 및 확장성

### MCP 서버 지원

Graphiti는 **Model Context Protocol (MCP) 서버** 구현을 포함하여 Claude, Cursor 등 AI 어시스턴트와의 직접 통합을 지원합니다. 이를 통해 기존 워크플로우를 변경하지 않고도 강력한 지식 그래프 기반 메모리 기능을 제공할 수 있습니다.[^9][^10]

### REST API 서비스

`server` 디렉토리에는 FastAPI 기반의 REST API 서비스가 포함되어 있어 Graphiti 기능을 웹 서비스로 노출할 수 있습니다. 이는 다양한 애플리케이션과의 통합을 용이하게 합니다.[^9]

### 오픈소스 커뮤니티

GitHub에서 **17.2k 스타**와 **1.5k 포크**를 기록하며 활발한 커뮤니티를 형성하고 있습니다. Apache-2.0 라이선스 하에 배포되어 상업적 활용도 가능하며, Zep Discord 서버의 \#Graphiti 채널을 통해 커뮤니티 지원을 받을 수 있습니다.[^9][^10][^11]

## 성능 최적화 및 확장성

### 병렬 처리 및 동시성 제어

Graphiti의 수집 파이프라인은 높은 동시성을 위해 설계되었으며, 환경 변수 `SEMAPHORE_LIMIT`를 통해 동시 작업 수를 제어할 수 있습니다. 기본값은 LLM 제공업체의 429 속도 제한 오류를 방지하기 위해 10으로 설정되어 있지만, 더 높은 처리량을 지원하는 제공업체의 경우 이 값을 증가시켜 성능을 향상시킬 수 있습니다.[^9]

### 대용량 데이터셋 처리

Graphiti는 **병렬 처리**를 통해 대용량 데이터셋 처리에 최적화되어 있으며, 이벤트의 연대기를 유지하면서도 대량 처리 시 LLM 호출의 병렬 처리를 지원합니다. Neo4j의 병렬 런타임 기능을 활용할 수 있는 `USE_PARALLEL_RUNTIME` 옵션도 제공됩니다(Neo4j Community 에디션이나 소규모 AuraDB 인스턴스에서는 지원되지 않음).[^5][^9][^11]

## 향후 개발 방향 및 로드맵

### 커스텀 그래프 스키마 지원

개발 팀은 개발자가 에피소드 수집 시 자체 정의된 노드 및 엣지 클래스를 제공할 수 있도록 하는 **커스텀 그래프 스키마** 지원을 계획하고 있습니다. 이를 통해 특정 사용 사례에 맞춘 더 유연한 지식 표현이 가능해질 것입니다.[^9]

### 검색 기능 강화

더 견고하고 구성 가능한 옵션으로 검색 기능을 강화하는 작업이 진행 중입니다. 이는 다양한 사용 사례와 요구사항에 대응하기 위한 것으로, 검색 정확도와 효율성의 추가적인 개선을 목표로 합니다.[^9]

### 테스트 커버리지 확대

신뢰성 확보와 엣지 케이스 감지를 위한 테스트 커버리지 확대 작업도 진행되고 있습니다. 이는 프로덕션 환경에서의 안정성을 보장하기 위한 중요한 개발 방향입니다.[^9]

## 제한사항 및 고려사항

### LLM 의존성

Graphiti는 엔티티 추출, 관계 해결, 시간적 추출 등 핵심 기능에서 LLM에 크게 의존합니다. 이는 API 비용과 처리 속도에 영향을 미칠 수 있으며, 특히 소규모 모델 사용 시 부정확한 출력 스키마와 수집 실패를 초래할 수 있습니다.[^1][^9]

### 초기 설정 복잡성

여러 구성 요소(그래프 데이터베이스, LLM 서비스, 임베딩 서비스)의 조합으로 인해 초기 설정이 복잡할 수 있습니다. 특히 엔터프라이즈 환경에서는 보안, 컴플라이언스, 인프라 요구사항을 고려한 배포 계획이 필요합니다.[^9]

## 결론

Graphiti는 전통적인 RAG 시스템의 한계를 극복하고 AI 에이전트의 동적 메모리 요구사항을 충족하는 혁신적인 솔루션입니다. 시간 인식 지식 그래프, 실시간 증분 업데이트, 하이브리드 검색 등의 핵심 기능을 통해 기존 접근법 대비 뛰어난 성능과 확장성을 제공합니다.[^1][^2][^3][^5]

특히 LongMemEval 벤치마크에서 최대 18.5%의 정확도 향상과 90%의 지연 시간 단축을 달성한 것은 실제 기업 환경에서의 적용 가능성을 강력하게 시사합니다. Apache-2.0 라이선스 하에 제공되는 오픈소스 특성과 활발한 커뮤니티 지원, 그리고 다양한 LLM 제공업체 및 데이터베이스 백엔드와의 호환성은 Graphiti를 실제 프로덕션 환경에 도입하기에 매력적인 선택지로 만듭니다.[^3][^8][^9][^11][^1]

향후 커스텀 그래프 스키마 지원과 검색 기능 강화 등의 개발 방향을 고려할 때, Graphiti는 AI 에이전트 메모리 분야에서 지속적인 혁신을 이끌어갈 것으로 예상됩니다.




[^1]: https://github.com/getzep/graphiti

[^2]: https://arxiv.org/html/2501.13956v1

[^3]: https://www.youtube.com/watch?v=PxcOIINgiaA

[^4]: https://mcpmarket.com/server/graphiti

[^5]: https://discuss.pytorch.kr/t/graphiti/5180

[^6]: https://www.linkedin.com/posts/vrinda-rani_rag-vs-graphrag-vs-graphiti-activity-7325827650810183681-dvMm

[^7]: https://neo4j.com/blog/developer/graphiti-knowledge-graph-memory/

[^8]: https://blog.getzep.com/graphiti-knowledge-graphs-for-agents/

[^9]: https://www.linkedin.com/pulse/semantic-showdown-graphrag-vs-graphiti-race-memory-dipanjan-chowdhury-dlzwc

[^10]: https://www.getzep.com/product/open-source/

[^11]: https://www.reddit.com/r/LocalLLaMA/comments/1hft9va/graphiti_temporal_knowledge_graph_with_local_llms/

[^12]: https://falkordb.com/blog/what-is-graphrag/

[^13]: https://www.youtube.com/watch?v=H2Cb5wbcRzo

[^14]: https://www.themoonlight.io/ko/review/zep-a-temporal-knowledge-graph-architecture-for-agent-memory

[^15]: https://arxiv.org/abs/2501.13956

[^16]: https://help.getzep.com/graphiti/getting-started/overview

[^17]: https://www.falkordb.com/blog/building-temporal-knowledge-graphs-graphiti/

[^18]: https://airbyte.com/data-engineering-resources/customer-data-integration

[^19]: https://graphrag.com/appendices/research/2501.13956/
[^20]: https://www.apideck.com/blog/25-crm-apis-to-integrate-with
[^21]: https://www.salesforce.com/ap/crm/crm-integration/
[^22]: https://blog.getzep.com/how-do-you-search-a-knowledge-graph/
[^23]: https://blog.getzep.com/state-of-the-art-agent-memory/
[^24]: https://neo4j.com/blog/from-zero-to-the-connected-enterprise-in-45-minutes
[^25]: https://www.reddit.com/r/LocalLLaMA/comments/1kavtwr/benchmarking_ai_agent_memory_providers_for/

