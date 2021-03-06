# tekton-polling-operator

A simple git repository poller.

This polls a GitHub repository, and triggers pipeline runs when the SHA of the
a specific ref changes.

It _does not_ use API tokens to do this, instead it uses the method documented
[here](https://developer.github.com/changes/2016-02-24-commit-reference-sha-api/)
and the ETag to fetch the commit.

## Installation

This operator requires Tekton Pipelines to be installed first, the installation
instructions are [here](https://github.com/tektoncd/pipeline/blob/master/docs/install.md).

```shell
$ kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.14.0/release.yaml
```

Then you'll need to install the polling-operator.

```shell
$ kubectl apply -f https://github.com/bigkevmcd/tekton-polling-operator/releases/download/v0.0.1/release-0.0.1.yaml
```

## Pipelines

You'll want a pipeline to be executed on change.

This pipeline **must** accept two parameters:

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: demo-pipeline
spec:
  params:
  - name: sha
    type: string
    description: "the SHA of the recently detected change"
  - name: repoURL
    type: string
    description: "the cloneURL that the change was detected in"
  tasks:
```

A sample pipeline  is provided in the [examples](./examples) directory.

## Monitoring a Repository

To monitor a repository for changes, you'll need to create a `Repository` object
in Kubernetes.

```yaml
apiVersion: polling.tekton.dev/v1alpha1
kind: Repository
metadata:
  name: example-repository
spec:
  url: https://github.com/my-org/my-repo.git
  ref: main
  frequency: 5m
  pipelineRef:
    name: github-poll-pipeline
```

This defines a repository that monitors the `main` branch in
`https://github.com/my-org/my-repo.git`, checking every 5 minutes, and executing
the `github-poll-pipeline` when a change is detected.

## Authenticating against a Private Repository

Of course, not every repo is public, to authenticate your requests, you'll
need to provide an auth token.

```yaml
apiVersion: polling.tekton.dev/v1alpha1
kind: Repository
metadata:
  name: example-repository
spec:
  url: https://github.com/my-org/my-repo.git
  ref: main
  frequency: 2m
  pipelineRef:
    name: github-poll-pipeline
  auth:
    secretRef:
      name:  my-github-secret
    key: token
```

This will fetch the secret, and get the value in `token` and use that to
authenticate the API call to GitHub.
