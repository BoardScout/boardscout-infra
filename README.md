# BoardScout Infrastructure

BoardScout의 6개 컨테이너를 Docker Compose로 실행합니다.

| 컨테이너 | 역할 | 외부 공개 |
| --- | --- | --- |
| `nginx` | 단일 진입점과 리버스 프록시 | `80` |
| `frontend` | 정적 HTML/CSS/JS 제공 | 없음 |
| `spring` | 핵심 API | 없음 |
| `fastapi` | 추천 API | 없음 |
| `redis` | 캐시 | 없음 |
| `mysql` | 영구 데이터베이스 | 없음 |

## 실행

```bash
cp .env.example .env
docker compose up --build
```

접속 주소는 `http://localhost`입니다. Nginx는 `/api/core/*`를 Spring으로, `/api/recommend/*`를 FastAPI로 전달하고 나머지 요청을 Frontend로 전달합니다.

## 추천 기능만 빠르게 테스트

프런트와 FastAPI 추천 기능만 확인할 때는 Spring, MySQL, Redis 없이 세 개의 컨테이너만 실행합니다.

```bash
docker compose -f compose.recommend.yaml up --build
```

브라우저에서 `http://localhost:8088`을 열고 `RIGHT NOW` 또는 `MY TASTE` 추천을 진행합니다.
API 연결만 확인하려면 다음 주소를 사용합니다.

```bash
curl http://localhost:8088/api/recommend/health

curl -X POST 'http://localhost:8088/api/recommend/games?limit=5' \
  -H 'Content-Type: application/json' \
  -d '{"recommendationMode":"taste","likedGameIds":[0,1,2]}'
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
