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

중지할 때는 다음 명령을 사용합니다.

```bash
docker compose down
```

MySQL과 Redis 데이터까지 초기화하려면 명시적으로 `docker compose down --volumes`를 실행합니다.
