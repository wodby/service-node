# Node.js service for Kubernetes on Wodby

Build and run Node.js applications on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Node.js.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Start with a template

Use one of the source templates exposed by this service to start with
compatible build configuration and Wodby CI:

- [Express.js boilerplate](https://github.com/wodby/expressjs-boilerplate)

## Wodby stacks using this service

- [Node.js application stack](https://github.com/wodby/stack-node)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `node` |
| Type | Application service |
| Versions | `26` by default; also available: `24`, `22`, `20` |
| Workloads | `main` (Deployment), primary; scalable |
| Containers | `node` using `wodby/node`, build target |
| Endpoints | `node`: HTTP 3000 (main) |
| Service links | DBMS (`db`), optional, Mail Transfer Agent (`sendmail`), optional, Redis, optional |
| Application build | Git source connection enabled; Dockerfile: `Dockerfile`; starters: Express.js boilerplate |
| Helm | chart `oci://registry-1.docker.io/wodby/node`; version `0.2.0` |

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
