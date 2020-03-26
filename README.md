# Snyk Orb for CircleCI

## The Snyk Orb

Use the Snyk orb to easily incorporate Snyk into your CircleCI Workflows.

By utilizing this orb in your project workflow, it is possible to use Snyk to test, fix and monitor your project for vulnerabilities in the app dependencies and Docker images, all with a single command. You can set thresholds for vulnerability tolerance in your app or Docker image (and fail the workflows when threshold is exceeded), apply proprietary Snyk patches, and save dependency snapshots on snyk.io for continuous monitoring and alerting.

## How to use the Snyk Orb

In fact, it is very easy to start using the Orb.
All you need to do is:

1. Follow the instructions at the [Orb Quick Start Guide](https://circleci.com/orbs/registry/orb/snyk/snyk#quick-start) to enable usage of Orbs in your project workflow.
2. Set up an environment variable (`SNYK_TOKEN`) with your Snyk API token, which you can get from your [account](https://app.snyk.io/account).
3. In the app build job, call the `snyk/scan`
4. Optionally, supply parameters to customize orb behaviour

## Usage Examples

### Scan App Dependencies

```yaml
version: 2.1

  orbs:
    snyk: snyk/snyk@x.y.z

  jobs:
    build:
      docker:
        - image: circleci/node:4.8.2
      steps:
        - checkout
        - run: npm install -q
        - snyk/scan
```

### Scan Docker Image

```yaml
version: 2.1

orbs:
  snyk: snyk/snyk@x.y.z

jobs:
  build:
    environment:
      IMAGE_NAME: myrepo/myapp
    docker:
        - image: circleci/buildpack-deps:stretch
    steps:
      - checkout
      - setup_remote_docker
      - run:
          name: Build Docker image
          command: docker build -t $IMAGE_NAME:latest .
      - snyk/scan:
          token-variable: SNYK_TOKEN
          docker-image-name: $IMAGE_NAME:latest
          target-file: "Dockerfile"
```

### Advanced Example

```yaml
version: 2.1

orbs:
snyk: snyk/snyk@x.y.z

jobs:
build:
    docker:
    - image: circleci/node:4.8.2
    steps:
    - checkout
    - run:
        command: npm install -q
    - snyk/scan:
        token-variable: CICD_SNYK_TOKEN                           # use is api token stored in an env variable named other than SNYK_TOKEN
        severity-threshold: high                                  # only fail if detected high-severity vulnerabilities
        protect: true                                             # apply pactches specified in commited .snyk file (generated by running the [snyk wizard](https://snyk.io/docs/cli-wizard/))
        fail-on-issues: false                                     # don't fail even if issues detected (not recommended!)
        monitor-on-build: true                                    # create a snapshot of apps dependencies on snyk.io, for continoues monitoring (recommended!)
        project: ${CIRCLE_PROJECT_REPONAME}/${CIRCLE_BRANCH}-app  # use this to save the snapshot under specific names.
        organization: ${SNYK_CICD_ORGANIZATION}                   # save reports under a specific Snyk organization
```

## Orb Parameters

Full reference docs https://circleci.com/orbs/registry/orb/snyk/snyk

| Parameter  | Description | Required | Default | Type |
| -----------| -------------------------------------------------------------------------------------------------------- | ------------- | ------------- | ------------- |
| token-variable | Name of env var containing your Snyk API token | no | SNYK_TOKEN | env_var_name |
| severity-threshold | Only report vulnerabilities of provided level or higher (low/medium/high) | no | low | low \| med \| high |
| protect | Protect the app by applying patches specified in your .snyk file (after running the Snyk wizard) | no | false | boolean |
| fail-on-issues | This specifies if builds should be failed or continued based on issues found by Snyk | no | true | boolean |
| monitor-on-build | Take a current application dependencies snapshot for continuous monitoring by Snyk, if test was succesful | no | true | boolean |
| target-file | The path to the manifest file to be used by Snyk. Should be provided if non-standard | no | - | string |
| docker-image-name | The image name, if scanning a container image | no | - | string |
| organization | Name of the Snyk organisation name, under which this project should be tested and monitored | no | - | string |
| project | A custom name for the Snyk project to be created on snyk.io | no | - | string |
| additional-arguments | Refer to the Snyk CLI help page for information on additional arguments | no | - | string |
| os | The CLI OS version to download | no | linux | linux \| macos \| alpine |
| install-alpine-dependencies | For the alpine CLI, should extenral dependencies be installed | no | true | boolean |

## Screenshots

### Snyk found high-severity vulnerabilies in app and failed the workflow

![Snyk Found Vulns](pictures/snyk_found_vulns.png)

### Continuous Monitoring on snyk.io

#### Issues Report

![Issues Report on snyk.io](pictures/snykio_report.png)

#### Dependency Tree

![Dependency Tree on snyk.io](pictures/snykio_deptree.png)
