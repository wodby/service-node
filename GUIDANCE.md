# Node.js on Wodby

What Wodby sets up for an application that runs on this service. Check it before changing the start command, the port or connection settings in the code.

## How the application is started

The image built by the service's Dockerfile starts the application with `npm run start`, from `/usr/src/app`. `package.json` must define a `start` script.

- The application must listen on port 3000, on all interfaces. That is the port of the service's HTTP endpoint. The service sets `NODE_PORT` to that number; it does not set `PORT`. Read `NODE_PORT`, or default to 3000.
- `NODE_ENV` is `development` in environments of type `dev` and `production` in every other type.

## Build

The service's Dockerfile only copies the build context to `/usr/src/app`. It does not run `npm install`. Dependencies, and any compiled output, must be produced by the pipeline before the image is built, for example with `wodby ci run -- npm ci`, so that `node_modules` is part of what is copied. An image built without `node_modules` cannot load the application's dependencies.

A pipeline that passes a Dockerfile from the repository (`wodby ci build node -f Dockerfile`) uses that file instead.

The service declares no volume of its own: files written inside the container are lost when it is replaced.

## Linked services

Links to other services reach the application as environment variables. Nothing in the image reads them: the application reads them itself. Do not hardcode hosts or credentials.

| Link | Variables |
| --- | --- |
| Database (MariaDB, MySQL or PostgreSQL) | `DB_HOST`, `DB_PORT`, `DB_NAME` (also `DB_DATABASE`), `DB_USERNAME`, `DB_PASSWORD`, `DB_DRIVER` (also `DB_CONNECTION`) |
| Mail | `SMTP_HOST`, `SMTP_PORT` |
| Redis or Valkey | `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD` |

All three links are optional. A variable is present only while its link exists and the linked service is enabled. The mail link carries no credentials.

## Environment

- `WODBY_HOSTS` is a JSON array of the environment's hosts, not a comma-separated string. `WODBY_PRIMARY_HOST` and `WODBY_PRIMARY_URL` are the canonical ones for links generated outside a request.
- `WODBY_APP_SERVICE_NAME` is this service's host name inside the environment. Other services reach the application at that name on port 3000.
- `WODBY_ENV_TYPE` tells a development environment from a production-like one.

## In a development workspace

- The checkout is mounted at `/usr/src/app` and served as it is on disk.
- Workspace setup installs dependencies with `workspace-node prepare`. The package manager is taken from `packageManager` in `package.json`, otherwise from the lockfile: `pnpm-lock.yaml` means pnpm, `yarn.lock` means Yarn, anything else npm. Installs follow the lockfile exactly (`npm ci`, `--frozen-lockfile`, `--immutable`), with development dependencies. Run it again after changing dependencies.
- The application is started with the `dev` script of `package.json` when there is one, otherwise with `start`. `WORKSPACE_NODE_COMMAND` on the service replaces that command.
- `NODE_ENV` is always `development` here, whatever the environment type. `PORT` is set, to `NODE_PORT` unless given, and `HOST` to `0.0.0.0`.
- Whether a saved file is picked up depends on the script. A script without a watcher, such as `node server.js`, needs a restart of the service. For scripts with a watcher, `CHOKIDAR_USEPOLLING`, `CHOKIDAR_INTERVAL` and `WATCHPACK_POLLING` are set, because a checkout on network storage needs polling. `WORKSPACE_POLL_INTERVAL` changes the interval, `WORKSPACE_POLLING=0` turns this off.
- `node_modules/` and `/.pnpm-store/` are kept out of Git status without touching `.gitignore`.
- A change to variables or linked services still needs a deployment of the environment.

## Check the result

- `curl -s -o /dev/null -w '%{http_code}' "localhost:$NODE_PORT"` from the container shows whether the application answers on the expected port.
- `printenv | grep -E '^(DB|SMTP|REDIS)_' | cut -d= -f1` lists the link variables that are present.
