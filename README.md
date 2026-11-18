# BoardScout Infrastructure

BoardScout의 7개 컨테이너를 Docker Compose로 실행합니다. 과제 제출용 `docker run` 절차는 보고서에 정리되어 있습니다.

| 컨테이너 | 역할 | 외부 공개 |
| --- | --- | --- |
| `boardscout-proxy` | 단일 진입점과 역방향 프록시(Nginx) | `80` |
| `boardscout-web` | 정적 HTML/CSS/JS 제공 | 없음 |
| `boardscout-core-1`, `boardscout-core-2` | 핵심 서버(Spring Boot) 2대, 회원·인증·카탈로그·별점 | 없음 |
| `boardscout-recommender` | 추천 서버(FastAPI) | 없음 |
| `boardscout-session` | 서버 간 공유 세션 저장소(Redis) | 없음 |
| `boardscout-db` | 영구 데이터베이스(MySQL) | 없음 |

## 실행

```bash
cp .env.example .env
docker compose up --build
```

접속 주소는 `http://localhost`입니다. 프록시는 `/api/v1/recommendations/*`를 추천 서버로, 그 밖의 `/api/v1/*`를 핵심 서버 2대에 번갈아(round-robin) 전달하고, 나머지 요청을 web으로 전달합니다. 응답 헤더 `X-Upstream`에서 요청을 처리한 서버 주소를 확인할 수 있습니다.

## 추천 기능만 빠르게 테스트

프런트와 FastAPI 추천 기능만 확인할 때는 Spring, MySQL, Redis 없이 세 개의 컨테이너만 실행합니다.

```bash
docker compose -f compose.recommend.yaml up --build
```

브라우저에서 `http://localhost:8088`을 열고 `RIGHT NOW` 또는 `MY TASTE` 추천을 진행합니다.
API 연결만 확인하려면 다음 주소를 사용합니다.

```bash
curl http://localhost:8088/api/v1/recommendations/health

curl -X POST 'http://localhost:8088/api/v1/recommendations/games?limit=5' \
  -H 'Content-Type: application/json' \
  -d '{"recommendationMode":"taste"}'
```

테스트를 마치면 컨테이너를 종료합니다.

```bash
docker compose -f compose.recommend.yaml down
```

중지할 때는 다음 명령을 사용합니다.

```bash
docker compose down
```

MySQL과 Redis 데이터까지 초기화하려면 명시적으로 `docker compose down --volumes`를 실행합니다.

## MySQL 통합 테스트

Spring 통합 테스트용 MySQL만 실행할 수 있습니다. 전체 스택의 내부 네트워크와 영구 데이터는 사용하지 않으며, 테스트 DB는 컨테이너 종료 시 초기화됩니다.

```bash
docker compose -f compose.test.yaml up -d --wait
cd ../spring-server && ./gradlew mysqlTest
cd ../infra && docker compose -f compose.test.yaml down
```

기본 호스트 포트는 `3307`이며 `MYSQL_TEST_PORT` 환경 변수로 변경할 수 있습니다.
