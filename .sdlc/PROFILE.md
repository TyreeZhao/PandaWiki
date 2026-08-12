# Project Profile: PandaWiki

tech-stack: [Go 1.24.3, Echo, GORM, PostgreSQL, Redis, NATS JetStream, MinIO, RAGLite, Eino, ModelKit, React 19, Vite 6, Next.js 16, TypeScript 5.9, pnpm 10]
test-commands: { unit: "cd backend && go test ./...", coverage: "cd backend && go test ./... -coverprofile=coverage.out", e2e: "none - repository has no dedicated e2e suite", typecheck: "cd web/admin && pnpm exec tsc -b; cd web/app && pnpm exec tsc --noEmit", build: "cd backend && go build ./...; cd web && pnpm build" }

## Tech stack

| Layer | Technology | Why it is present |
|---|---|---|
| Backend API | Go 1.24.3, Echo, Wire, Viper | Implements the API, dependency injection, configuration, and long-running services |
| Domain and persistence | GORM, PostgreSQL, Redis | Stores domain data and provides cache/session/distributed state |
| Async processing | NATS JetStream, Go consumer | Runs document vectorization, summarization, crawling, and status synchronization |
| Object storage | MinIO/S3 | Stores uploaded documents and static files |
| AI and retrieval | RAGLite, Eino, ModelKit | Provides document parsing, retrieval, prompt/message composition, and model access |
| Admin frontend | React 19, Vite 6, Redux Toolkit, MUI | Provides the knowledge-base administration console |
| User frontend | Next.js 16 App Router, React 19 | Provides public knowledge-base pages, authentication, widget, and chat experiences |
| Shared frontend | pnpm workspace packages | Shares icons, themes, and UI components between frontend applications |
| SDK | Independent Go RAG module | Wraps dataset, document, chunk, and retrieval HTTP APIs |

## Surface map

- backend-api: globs[ backend/cmd/api/**, backend/server/**, backend/handler/v1/**, backend/handler/share/**, backend/middleware/**, backend/api/** ] roles[server-dev, qa] modes[correctness, e2e:OpenAPI]
- backend-domain: globs[ backend/domain/**, backend/consts/**, backend/usecase/**, backend/repo/**, backend/store/cache/**, backend/store/s3/**, backend/pkg/**, backend/utils/**, backend/setup/**, backend/log/**, backend/apm/**, backend/telemetry/** ] roles[server-dev, qa] modes[correctness]
- ai-rag: globs[ backend/usecase/chat.go, backend/usecase/llm.go, backend/domain/llm.go, backend/store/rag/**, backend/repo/pg/prompt.go, backend/domain/prompt.go, sdk/rag/** ] roles[server-dev, qa] modes[correctness, eval-bench]
- async-worker: globs[ backend/cmd/consumer/**, backend/cmd/migrate/**, backend/handler/mq/**, backend/mq/**, backend/migration/** ] roles[server-dev, qa] modes[correctness]
- data-schema: globs[ backend/store/pg/**, backend/**/*.sql ] roles[big-data, server-dev, qa] modes[correctness]
- professional-edition: globs[ backend/pro/**, backend/*pro*, backend/Dockerfile.*.pro ] roles[server-dev, qa] modes[correctness, e2e:OpenAPI]
- admin-web: globs[ web/admin/** ] roles[client-dev, design, qa] modes[correctness, e2e:Web]
- public-web: globs[ web/app/** ] roles[client-dev, design, qa] modes[correctness, e2e:Web]
- shared-web: globs[ web/packages/**, web/package.json, web/pnpm-workspace.yaml, web/tsconfig.base.json, web/prettier.config.js ] roles[client-dev, design, qa] modes[correctness, e2e:Web]
- generated-contracts: globs[ backend/docs/**, backend/cmd/**/wire_gen.go, web/admin/src/request/**, web/app/src/request/** ] roles[server-dev, client-dev, qa] modes[correctness, e2e:OpenAPI]
- project-infra: globs[ .github/**, AGENTS.md, README.md, PROJECT_STRUCTURE.md, backend/Makefile, backend/Dockerfile*, web/**/Dockerfile, web/**/Makefile, .gitignore, web/.gitignore ] roles[server-dev, qa, ai-readiness] modes[correctness]
- docs-assets: globs[ images/**, CODE_OF_CONDUCT.md, CONTRIBUTING.md, SECURITY.md, LICENSE ] roles[qa] modes[correctness]

## Conventions

- Communicate in Chinese unless the user explicitly requests another language.
- Prefer small, targeted changes that follow nearby repository patterns.
- Keep backend business logic in `backend/usecase/`, HTTP-specific behavior in `backend/handler/`, persistence and external access in `backend/repo/` or `backend/store/`, and shared models in `backend/domain/` or `backend/api/`.
- Use `gofmt` and `goimports`; wrap errors with `%w`, use `errors.Is`, and preserve context propagation.
- Use `Req` and `Resp` suffixes for backend request and response structs.
- Use pnpm only for frontend dependency and script execution.
- Follow TypeScript strict mode, existing path aliases, Prettier single quotes, and explicit types; avoid new `any` where practical.
- Do not hand-edit generated Wire, Swagger, or frontend request client files. Regenerate them using the owning command.
- Preserve SSE event shapes and cleanup behavior when changing streaming flows.
- A backend code change requires affected-package tests and `cd backend && make lint` before commit; `make lint` requires the professional-edition submodule.
- Admin verification uses targeted ESLint and build; App verification uses lint/typecheck/build because no frontend test suite exists.

## Entry points

- Backend API: `cd backend && go run cmd/api/main.go cmd/api/wire_gen.go`; production image starts `/app/panda-wiki-migrate && /app/panda-wiki-api`.
- Backend consumer: `cd backend && go run cmd/consumer/main.go cmd/consumer/wire_gen.go`.
- Database migration: `cd backend && go run cmd/migrate/main.go cmd/migrate/wire_gen.go`.
- Backend dependency assembly: `backend/cmd/*/wire.go` and generated `wire_gen.go`.
- Management API routes: constructors under `backend/handler/v1/`, mounted under `/api/v1`.
- Public/chat routes: constructors under `backend/handler/share/`, mounted under `/share/v1`.
- Admin frontend: `cd web/admin && pnpm dev`; entry `web/admin/src/main.tsx`, route table `web/admin/src/router.tsx`.
- User frontend: `cd web/app && pnpm dev`; entry `web/app/src/app/layout.tsx`, proxy behavior `web/app/src/proxy.ts`.
- Whole frontend workspace: `cd web && pnpm dev` or `cd web && pnpm build`.
- Backend generation: `cd backend && make generate`; professional generation: `cd backend && make generate_pro`.
- Frontend API generation: `cd web/admin && pnpm api` and `cd web/app && pnpm api`.

## Known risks

- AI-readiness health is 4.4/10: type foundations are present, but testing, context hierarchy, local tooling, and automated safety gates are weak.
- `AGENTS.md` contains stale statements: it says no `AGENTS.md` exists and a root `CLAUDE.md` exists, while the repository has `AGENTS.md` and no `CLAUDE.md`.
- `AGENTS.md` references `cd web && pnpm api`, but the root web package has no `api` script; API clients are generated per application.
- `backend/pro` is an uninitialized Git submodule. Professional generation, professional builds, and the required backend `make lint` may fail until it is initialized.
- Test coverage is sparse: only a few backend packages have tests, frontend applications have no dedicated test suite, and backend CI does not run `go test ./...`.
- Repository guidance mentions an 80% coverage expectation, but no enforced coverage gate or matching test command exists.
- Sensitive-material review found a tracked private key, a source value shaped like an API token, and access-token logging sites. Treat these as security findings requiring separate remediation and secret rotation assessment.
- File and external-input paths include TLS verification bypass, fail-open extension policy behavior, raw SQL construction, URL fetching, local file reads, webhooks, and multipart input; changes in these areas require security-focused review.
- Large hotspots include `backend/repo/pg/node.go`, `backend/usecase/app.go`, `backend/usecase/node.go`, `web/admin/src/components/System/component/ModelConfig.tsx`, and duplicated large AI chat components.
- Generated Swagger clients and docs create substantial diff noise and should be regenerated rather than manually merged.
- Local Node and pnpm versions differ from CI and project declarations, which can produce inconsistent frontend results.
- Existing repository Git hooks are Git LFS hooks; replacing them outright would break LFS behavior. SDLC hooks must be composed with the existing scripts.

## AI-readiness assessment

| Dimension | Score | Evidence |
|---|---:|---|
| Context hierarchy | 2/10 | One root `AGENTS.md`, no `CLAUDE.md`, no domain-level context files |
| Context quality | 5/10 | Useful architecture guidance exists, but several statements are stale |
| Scoped commands | 6/10 | Backend package commands and separate frontend builds exist; shared-package checks are limited |
| Domain-aware checks | 5/10 | CI separates backend and web but does not route checks from the exact diff surface |
| Noise control | 5/10 | Common artifacts are ignored; generated clients/docs remain tracked and noisy |
| Type system | 7/10 | Go and TypeScript strict mode are present; explicit `any` remains common |
| Test readiness | 2/10 | Sparse backend tests, no frontend tests, no CI test gate |
| LSP readiness | 3/10 | Project metadata exists, but expected local language-server tools were not detected |
| Documentation | 6/10 | README and project structure exist, but testing and agent instructions have drifted |
| Prohibitions and guards | 3/10 | Layering rules exist; secret scanning, execution guards, and test gates are absent |

## Deploy

- target-type: container
- config locations: backend and frontend Dockerfiles; `.github/workflows/backend.yml`, `.github/workflows/backend_check.yml`, `.github/workflows/web.yml`
- runtime composition: not stored in this repository; README delegates installation to an external `manager.sh`
- image targets: API, consumer, admin Nginx, and Next.js app; professional backend images use `.pro` Dockerfiles
- environments: CI builds images for pull requests and pushes release-tagged images; repository does not define dev/staging/canary/full deployment coordinates
- secret source: GitHub Actions secrets and runtime environment variables; values must not be copied into repository documentation

## Evolution log

> Append-only project evolution history lives in `.sdlc/EVOLUTION.md`. This profile remains a bounded architecture snapshot.
