# Startup, setup, and deployment

Sub2API has two startup paths. On a fresh install it runs setup first. On an
already configured install it loads the full application and starts the HTTP
server plus background services.

```text
process start
  |
  +-- --version? print version and exit
  |
  +-- --setup? run CLI setup and exit
  |
  +-- needs setup?
        |
        +-- AUTO_SETUP=true -> create config/admin from env, then continue
        +-- otherwise       -> start setup wizard server and stop
  |
  +-- normal server startup
```

## Fresh install flow

1. The server checks the data directory. `DATA_DIR` wins, then writable
   `/app/data`, then the current directory.
2. Setup is needed only when both `config.yaml` and `.installed` are missing.
3. The setup flow tests PostgreSQL and Redis before writing configuration.
4. The database may be created if it does not already exist.
5. Setup creates the first admin only when the database has no users.
6. Setup writes `config.yaml` and `.installed`.
7. After setup, normal startup loads the app graph.

## Deployment choices

The repository supports three common deployment paths:

- Script install: downloads a release binary, installs under `/opt/sub2api`,
  creates a systemd service, then expects the setup wizard to finish config.
- Docker Compose: runs Sub2API with PostgreSQL and Redis containers and stores
  config/data in mounted directories.
- Source build: builds the Vue frontend, embeds it into the Go backend with the
  correct build tag, and runs the resulting server binary.

## Important rules

- Deleting data directories such as `data/`, `postgres_data/`, or `redis_data/`
  is destructive.
- If the backend is built without embedded frontend assets, the backend will not
  serve the web UI.
- In production simple mode, the config requires an explicit confirmation flag.
- Setup routes are blocked after installation.

## Source map

- `backend/cmd/server/main.go`
- `backend/internal/setup/setup.go`
- `backend/internal/setup/handler.go`
- `deploy/README.md`
- `deploy/docker-compose.local.yml`
- `README.md`

