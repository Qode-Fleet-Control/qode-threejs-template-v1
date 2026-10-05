# Three.js template

Provisioned from [`Qode-Fleet-Control/fleet-template-v1`](https://github.com/Qode-Fleet-Control/fleet-template-v1) — the fleet
lifecycle contract (`bin/`, `fleet.conf`, deploy workflows, `compose.yaml`) with a three.js scene (lit, rotating cube, grid, OrbitControls) bundled by Vite and served as a static site laid on top.

Listens on `0.0.0.0:$PORT` (default `3000`) and serves at the root (`/`) of its own hostname
(`https://<hash>.<FLEET_APP_DOMAIN>/`); the health check hits `/`. In the container: the static build (`dist/`) behind nginx.

## Origin

    hand-written to the three.js manual's Installation page (option 1: npm install --save three; npm install --save-dev vite; index.html + main.js) and its "Creating a scene" example

Generated 2026-10-05 with three 0.186.1, vite 8.3.2 (host Node v22.12.0 / npm 10.9.0).

## Run it

### On the fleet

The fleet clones the repo, injects `PORT` (and the workspace's `DATABASE_URL`, `REDIS_URL`, ...) and runs
`bin/run`, which uses the docker runtime from `fleet.conf`: `docker compose build`, then `docker compose up --remove-orphans` in the foreground.

### With docker

    PORT=3000 bin/run                  # what the fleet does
    docker compose up --build        # or plain compose

### Without docker

`FLEET_RUNTIME=process bin/run` runs the plain commands from `fleet.conf`:

| step | command |
|---|---|
| install | `npm install` |
| build | `npm run build` |
| start | `npx vite preview --host 0.0.0.0 --port $PORT` |

    ./bin/run       # install, build, start in the foreground
    ./bin/start     # start from existing build artifacts
    ./bin/restart   # rebuild and restart
    ./bin/stop      # stop whatever holds the port

See `docs/fleet-lifecycle.md` for the full contract.

## Deviations from the generator output

- `main.js` extends the manual's cube with lights, `MeshStandardMaterial`, a grid and `OrbitControls` from `three/addons`.
- `vite.config.js` added: `vite.config.js` sets `server.allowedHosts` / `preview.allowedHosts` from `FLEET_APP_HOST` (any host when unset): Vite otherwise answers the fleet hostname with "Blocked request" in `vite dev` / `vite preview`. It also raises `build.chunkSizeWarningLimit` (three.js alone is ~550 kB).
- Added the fleet files: `bin/` (lifecycle scripts), `fleet.conf`, `Dockerfile`, `compose.yaml`, `.dockerignore`, `.env.example`, `.github/workflows/`, `docs/fleet-lifecycle.md`; fleet entries (`.fleet/`, `*.log`, ...) prepended to `.gitignore`.

## Verified

**Not yet verified in docker.** On 2026-10-05 the shared docker host's disk stayed at 0-2 GB free for over 3 hours (held by other workloads), so the image was never built; `verify.sh` / `docker compose run` must still be run before this is trusted. `migrate.py audit`: READY. Without docker: `npm run build` (vite 8) succeeded.
