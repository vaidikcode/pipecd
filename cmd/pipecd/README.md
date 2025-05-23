# PipeCD Controle Plane
## Development

## Prerequisites

- [Go 1.24 or later](https://go.dev/)
- [NodeJS v20 or later](https://nodejs.org/en/)
- [Docker](https://www.docker.com/)
- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) (If you want to run Control Plane locally)
- [helm 3.8](https://helm.sh/docs/intro/install/) (If you want to run Control Plane locally)

## Repositories
- [pipecd](https://github.com/pipe-cd/pipecd): contains all source code and documentation of PipeCD project.

## Commands

- `make build/go`: builds all go modules including pipecd, piped, pipectl.
- `make test/go`: runs all unit tests of go modules.
- `make test/integration`: runs integration tests.
- `make gen/code`: generate Go and Typescript code from protos and mock configs. You need to run it if you modified any proto or mock definition files.

For the full list of available commands, please see the Makefile at the root of repository.

## How to run Control Plane locally

1. Start running a Kubernetes cluster (If you don't have any Kubernetes cluster to use)

    ``` console
    make kind-up
    ```

    Once it is no longer used, run `make kind-down` to delete it.

2. Install the web dependencies module

    ``` console
    make update/web-deps
    ```

3. Install Control Plane into the local cluster

    ``` console
    make run/pipecd
    ```

    Once all components are running up, use `kubectl port-forward` to expose the installed Control Plane on your localhost:

    ``` console
    kubectl -n pipecd port-forward svc/pipecd 8080
    ```

4. Access to the local Control Plane web console

    Point your web browser to [http://localhost:8080](http://localhost:8080) to login with the configured static admin account: project = `quickstart`, username = `hello-pipecd`, password = `hello-pipecd`.

## How to run Piped locally and add an application to your cluster
See [How to run Piped agent locally](https://github.com/pipe-cd/pipecd/tree/master/cmd/piped#how-to-run-piped-agent-locally).


##### Testing OIDC with Keycloak

PipeCD provides a built-in tool to quickly set up a local Keycloak server for testing OIDC integration:

```bash
make setup-oidc
```

This command:
1. Launches a Keycloak container with Docker
2. Creates a pre-configured realm, client, and test users with appropriate roles
3. Sets up protocol mappers to expose groups and username claims
4. Outputs the configuration you need to set up PipeCD

**Test Users**

The setup creates three test users with different roles:

| Username | Password | Role |
|----------|----------|------|
| admin-user | password | admin |
| editor-user | password | editor |
| viewer-user | password | viewer |

**Control Plane Configuration**

After running the setup, you'll receive a configuration similar to:

```yaml
apiVersion: "pipecd.dev/v1beta1"
kind: ControlPlane
spec:
  stateKey: "random-string-for-oauth-security" # Required: Use any value
  sharedSSOConfigs:
    - name: oidc
      provider: OIDC
      oidc:
        clientId: "pipecd-client"
        clientSecret: "pipecd-client-secret"
        issuer: "http://localhost:8080/realms/pipecd-test"
        redirectUri: "http://localhost:8080/auth/callback"
        scopes:
          - openid
          - profile
```

Start or restart your Control Plane with the above config.

**Managing the Keycloak Instance**

- Access the admin console at: http://localhost:8080/admin (admin/admin)
- Stop the container: `docker stop pipecd-keycloak`
- Remove the container: `docker rm pipecd-keycloak`

