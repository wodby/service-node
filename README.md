# Node.js service for Kubernetes on Wodby

Build and run Node.js applications on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Node.js.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Start with a boilerplate

Use one of the boilerplates exposed by this service to start with compatible
build configuration and Wodby CI:

- [Express.js boilerplate](https://github.com/wodby/expressjs-boilerplate)

## Wodby stacks using this service

- [Node.js application stack](https://github.com/wodby/stack-node)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `node` |
| Type | Application service |
| Versions | `26` by default; also available: `24`, `22` |
| Workloads | `main` (Deployment), primary; scalable |
| Containers | `node` using `wodby/node`, build target |
| Endpoints | `node`: HTTP 3000 (main) |
| Service links | DBMS (`db`), optional, Mail Transfer Agent (`sendmail`), optional, Redis, optional |
| Application build | Git source connection enabled; Dockerfile: `Dockerfile`; boilerplates: Express.js boilerplate |
| Helm | chart `oci://registry-1.docker.io/wodby/node`; version `0.3.2` |

## Use this service

Use this service through [Node.js application stack](https://github.com/wodby/stack-node), or reference `node` from a custom
Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).

## Development workspaces

Workspace support uses the development image and a persistent checkout at
`/usr/src/app`. Preparation installs development dependencies with npm, Yarn, or
pnpm, using the repository's `packageManager` field or lockfile. Pin Yarn/pnpm in
`packageManager` for reproducible setup.

Startup runs the project's `dev` script, falling back to `start`. Set the service
environment variable `WORKSPACE_NODE_COMMAND` to override that command, for example
`workspace-node vite-start` for a Vite project. The
command executes in the repository root. Dependencies are installed on preparation,
not on every restart. Projects without a watcher need an application service restart
after edits; restarting the SSH runner does not restart the application.

The server must listen on `0.0.0.0:3000`. The runtime exports `PORT=3000` and
`HOST=0.0.0.0` by default, but frameworks may require explicit command flags:

| Project | Example workspace command |
| --- | --- |
| Next.js | `workspace-node next-start` |
| React or Vue with Vite | `workspace-node vite-start` |
| Angular | `workspace-node angular-start` |

Allow the actual preview hostname in the framework's development-server configuration.
For Vite, `__VITE_ADDITIONAL_SERVER_ALLOWED_HOSTS` adds explicit hostnames. Preserve
host checks. Test live reload through the preview's HTTPS/WebSocket connection;
network filesystems may require project-specific polling configuration.

These frameworks share the Node runtime; they do not require separate SSH services.
Production builds and their startup commands remain separate from workspace startup.

Workspace-capable runtime images must declare `com.wodby.workspace.contract=1`.
The runtime defaults Chokidar/Watchpack to 1000 ms polling; configure other watchers
explicitly. Polling costs CPU, so exclude dependencies and generated build output.
For Next.js shared-volume development use `workspace-node next-start`, or a custom
Webpack-based command with polling. Generic Node continues to advertise restart
semantics because arbitrary project scripts may not have a watcher.

The framework helpers run the installed framework directly, bypassing custom npm
lifecycle scripts. `vite-start` loads the project's Vite config and enables both
Chokidar and Rolldown polling without modifying config files. `angular-start` passes
`--poll` to the Angular CLI. Use a custom command if additional project setup is
needed, with equivalent polling configured explicitly.
