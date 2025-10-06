<p align="center">
  <img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" />
</p>

<p align="center">
  <b>Wakttu Backend</b> · 실시간 멀티플레이 미니게임 플랫폼 백엔드
</p>

<p align="center">
  <a href="https://yarnpkg.com" target="_blank"><img alt="yarn" src="https://img.shields.io/badge/yarn-1.x-2C8EBB?logo=yarn&logoColor=white"></a>
  <img alt="node" src="https://img.shields.io/badge/node-20%2B-339933?logo=node.js&logoColor=white">
  <img alt="nestjs" src="https://img.shields.io/badge/NestJS-10-EE2A5A?logo=nestjs&logoColor=white">
  <img alt="prisma" src="https://img.shields.io/badge/Prisma-5-2D3748?logo=prisma&logoColor=white">
  <img alt="postgres" src="https://img.shields.io/badge/PostgreSQL-15-4169E1?logo=postgresql&logoColor=white">
  <img alt="redis" src="https://img.shields.io/badge/Redis-7-D82C20?logo=redis&logoColor=white">
  <img alt="socket.io" src="https://img.shields.io/badge/Socket.IO-4-010101?logo=socketdotio&logoColor=white">
  <img alt="docker" src="https://img.shields.io/badge/Docker-ready-0db7ed?logo=docker&logoColor=white">
</p>

### 소개
- 실시간 게임(끝말잇기, 쿵쿵따, 골든벨, 음악 퀴즈, Cloud 등)을 제공하는 멀티플레이 백엔드입니다.
- 세션 기반 인증과 Redis/Mongo 세션 스토어, Socket.IO 네임스페이스(`wakttu`)를 사용합니다.
- Prisma + PostgreSQL로 도메인 모델을 관리하고, Swagger로 API 문서를 제공합니다.

### 주요 기능
- **인증/세션**: 로컬 로그인/회원가입, 외부(OAuth 유사) 흐름, 게스트 로그인, `express-session` 기반 세션 유지
- **소켓 실시간**: 방 생성/입장/퇴장, 팀 선택, 게임 진행(라운드/턴/채팅/이모티콘), 복구 및 재접속 처리
- **게임 모드**: 끝말잇기(`last`), 쿵쿵따(`kung`), 골든벨(`bell`), 음악퀴즈(`music`), Cloud(`cloud`) 지원 및 연습 모드(bot 포함)
- **레이트 리밋/연결 보호**: 커스텀 `EnhancedSessionAdapter`로 IP/글로벌 단위 연결 시도 제어
- **정적 리소스 제공**: `client/` 정적 페이지 서빙
- **문서화**: Swagger UI(`/api`)

### 기술 스택
- **Runtime/Language**: Node.js 20+, TypeScript
- **Framework**: NestJS 10 (WebSocket, Swagger, ServeStatic)
- **DB/ORM**: PostgreSQL, Prisma 5 (`prisma/schema.prisma`)
- **Cache/Session**: Redis 7, connect-redis (prod), connect-mongodb-session (dev)
- **Real-time**: Socket.IO 4 (namespace: `wakttu`)
- **Infra**: Dockerfile, docker-compose (Postgres/Redis/Backend)

### 폴더 구조 하이라이트
```
src/
  auth/         # 로그인/회원가입/외부 인증, 가드
  socket/       # WebSocket Gateway(Service/Guard)
  room/         # 방 CRUD 및 DTO/Entity
  last|kung|bell|music|cloud/ # 게임 로직 서비스 모듈
  prisma/       # PrismaModule/Service
  dictionary/   # 사전/퀴즈 데이터 API
  stats|item|music|user/ # 도메인 모듈들
  session.adapter.ts # 연결 제한 포함 커스텀 IoAdapter
  prisma.filter.ts   # Prisma 예외 필터
```

### 빠른 시작
```bash
yarn install
yarn build
yarn start:dev
```

서버는 기본적으로 `PORT` 환경변수를 사용하며, 미설정 시 3000에서 시작합니다. Swagger UI는 `/api` 경로에서 확인 가능합니다.

### 환경변수
아래는 대표 변수 예시입니다. 실제 키는 운영 환경에 맞게 설정하세요.

```
# 공통
PORT=3000
NODE_ENV=development|production|jogong
SECRET=your-session-secret

# Redis (prod에서 사용)
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0

# 세션(Mongo) - development에서 사용
SESSION_DB_URI=mongodb://localhost:27017
SESSION_DB_NAME=wakttu
SESSION_DB_COLLECTION=sessions

# DB
DATABASE_URL=postgresql://user:password@localhost:5432/wakttu

# Auth
REDIRECT_URL=http://localhost:3000
```

### Docker / Compose
`docker-compose.yml`이 Postgres, Redis, Backend를 함께 띄웁니다.

```bash
# 환경파일 준비 (postgres.env, redis.env, backend.env)
docker compose up -d
```

- Backend 이미지: `ghcr.io/wakttu/wakttu_backend:latest`
- 포트 매핑: `backend 1945:1945`, `postgres 5432:5432`, `redis 6379:6379`
- 네트워크: `wakttu` (bridge), `nginx-bridge`(external)

### Swagger
- URL: `/api`
- 설정: `src/main.ts`의 `DocumentBuilder`

### Socket.IO 사용 예시 (namespace: `wakttu`)
```ts
import { io } from 'socket.io-client';
const socket = io('http://localhost:3000/wakttu', { withCredentials: true });

socket.on('connected', () => console.log('connected'));
socket.emit('roomList');
socket.on('roomList', (list) => console.log(list));

// 방 생성
socket.emit('createRoom', { title: '방 제목', total: 4, type: 0, option: [] });

// 채팅
socket.emit('chat', { roomId: 'ROOM_ID', chat: '안녕' });
socket.on('chat', (payload) => console.log(payload));
```

주요 이벤트: `createRoom`, `enter`, `exit`, `ready`, `start`, `chat`,
게임별: `last.*`, `kung.*`, `bell.*`, `music.*`, `cloud.*`

### Prisma 모델(발췌)
`prisma/schema.prisma`에 사용자/방/사전/퀴즈/아이템/이모지/통계 등 모델이 정의되어 있습니다.

```prisma
model User { id String @id name String score Int @default(0) provider String ... }
model Room { id String @id @default(uuid()) title String users User[] ... }
model Dictionary { id String @id @map("_id") @@map("wakttu_ko") }
```

### 스크립트
```bash
yarn start         # Nest start
yarn start:dev     # watch 모드
yarn build         # prisma generate + build
yarn test          # unit tests
yarn test:e2e      # e2e tests
```

### 라이선스
본 저장소의 소스는 사내/개인 프로젝트 용도로 사용됩니다. 외부 배포 정책은 별도 문의 바랍니다.

