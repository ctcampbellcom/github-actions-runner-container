# Docker Container for GitHub Actions Runner

This project will build a Docker container with the specified version of the GitHub Actions Runner installed into it. Images are built for:

- Ubuntu
- RedHat UBI
- Debian

The container will run as an ephemeral runner, which is deregistered after a single job has completed. You could use this container to spin up short lived runners in response to GitHub sending a webhook when an Actions job is queued.

The container is configured with a `CMD` start point which runs registration, the runner service, and deregistration. As this is not an `ENTRYPOINT` the container is also suitable for use in [Actions Runner Controller](https://github.com/actions/actions-runner-controller/tree/master).

## Building the container

You can build this container using the following command:

```bash
$ docker build -f [debian-actions-runner|ubuntu-actions-runner|etc.]/Dockerfile -t <container_tag> .
```

There are some configurable build arguments that you can pass in to modify the container build:

* `BASE`: default value depends on the variant, but can be modified to specify an alternative base container image to start from
* `GH_RUNNER_VERSION`: default value `2.309.0` but can be used to specify an alternative version of the GitHub Actions runner
* `USER`: default value `runner`

## Running the container

The container image supports a number of environment variables that you can pass to the container to control the registration of the self hosted runner with GitHub.

When registering the runner you have three options for the type of runner that you are wanting to create, enterprise, organization or repository self-hosted runner.

You need to provide one of the following environment variable URLs which allow the runner to be registered:

* `RUNNER_ENTERPRISE_URL`: The url for enterprise when registering a enterprise runner; e.g. https://github.com/enterprises/<enterprise_account_name>
* `RUNNER_ORGANIZATION_URL`: The url for organization when registering an organization runner; e.g. https://github.com/octodemo
* `RUNNER_REPOSITORY_URL`: The url for the repository when registering a repository; e.g. https://github.com/peter-murray/node-hue-api

A GitHub Personal Access Token is required so that it can be used to obtain a short lived access token for the runner to register with GitHub. The permissions required on the Personal Access Token will depend upon to the use case of the token;

* enterprise runner: `admin:enterprise`
* organization runner: `admin:org`
* repository runner: `repo`

The token needs to be provided as the environment variable `GITHUB_TOKEN`.

### Optional environment variables:

* `GITHUB_SERVER`: The url for GHES server (if not connecting to `github.com`)
* `RUNNER_NAME`: The name for the runner, must be unique if not specified will use the hostname of the container.
* `RUNNER_LABELS`: A comma separated list of labels to associate with the runner over the default values. e.g. `tester,container-runner,production`
* `RUNNER_GROUP`: A runner group to associate the runner with in the organization or enterprise. If not specified will use the `default` group.

### Running using Docker commandline examples

1. Registering an Enterprise Runner;

    ```bash
    $ docker run -d \
      -e RUNNER_ENTERPRISE_URL=https://github.com/enterprises/octodemo \
      -e GITHUB_TOKEN=<PAT_token_with_admin:enterprise> \
      <container_name>
    ```

1. Registering an Organization Runner;

    ```bash
    $ docker run -d \
      -e RUNNER_ORGANIZATION_URL=https://github.com/octodemo \
      -e GITHUB_TOKEN=<PAT_token_with_admin:org> \
      <container_name>
    ```

1. Registering an Repository Runner;

    ```bash
    $ docker run -d \
      -e RUNNER_REPOSITORY_URL=https://github.com/octodemo/demo-repo \
      -e GITHUB_TOKEN=<PAT_token_with_repo access> \
      <container_name>
    ```
### Populating the runner tool cache

You can mount a pre-populated tool cache volume or bind mount to `/home/runner/_work/_tool` and this will be picked up by any Actions that cache their associated binaries. See [the docs](https://docs.github.com/en/enterprise-server@3.10/admin/github-actions/managing-access-to-actions-from-githubcom/setting-up-the-tool-cache-on-self-hosted-runners-without-internet-access) for advice on how to create this cache.
