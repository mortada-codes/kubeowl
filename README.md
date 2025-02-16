# KubeOwl 🦉

KubeOwl is a Kubernetes operator that seamlessly synchronizes environment variables between GitHub Actions and Kubernetes clusters, making secrets and configuration management effortless.

## Description

Managing environment variables and secrets across GitHub Actions and Kubernetes clusters can be challenging and error-prone. KubeOwl solves this by providing:

- 🔄 Automatic synchronization of environment variables from GitHub Actions to Kubernetes
- 🔐 Secure handling of sensitive data and secrets
- 📦 Support for multiple deployment environments (dev, staging, prod)
- 🎯 Selective synchronization based on namespaces and labels
- 🔍 Audit logging of all synchronization events

## Getting Started

### Prerequisites
- go version v1.23.0+
- docker version 17.03+
- kubectl version v1.11.3+
- Access to a Kubernetes v1.11.3+ cluster
- GitHub repository with Actions enabled
- GitHub Personal Access Token with appropriate permissions

### To Deploy on the cluster
**Build and push your image to the location specified by `IMG`:**

```sh
make docker-build docker-push IMG=<some-registry>/kubeowl:tag
```

**NOTE:** This image ought to be published in the personal registry you specified.
And it is required to have access to pull the image from the working environment.
Make sure you have the proper permission to the registry if the above commands don’t work.

**Install the CRDs into the cluster:**

```sh
make install
```

**Deploy the Manager to the cluster with the image specified by `IMG`:**

```sh
make deploy IMG=<some-registry>/kubeowl:tag
```

> **NOTE**: If you encounter RBAC errors, you may need to grant yourself cluster-admin
privileges or be logged in as admin.

### Configure and Deploy

1. Create a KubeOwl Configuration:

```yaml
apiVersion: sync.kubeowl.io/v1alpha1
kind: EnvSync
metadata:
  name: my-app-sync
spec:
  source:
    github:
      repository: owner/repo
      environment: production
  target:
    namespace: my-namespace
    secretName: my-app-secrets
```

2. Configure GitHub Actions:

```yaml
name: Sync Environment Variables

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  sync-env:
    runs-on: ubuntu-latest
    steps:
      - name: Configure KubeOwl
        uses: kubeowl/action@v1
        with:
          cluster-url: ${{ secrets.CLUSTER_URL }}
          token: ${{ secrets.KUBEOWL_TOKEN }}
          namespace: my-namespace
```

3. Monitor Synchronization:

```sh
kubectl get envsyncs
kubectl describe envsync my-app-sync
```

### To Uninstall

1. Remove EnvSync resources:
```sh
kubectl delete envsyncs --all
```

2. Uninstall KubeOwl:
```sh
make undeploy
```

## Contributing

We welcome contributions to KubeOwl! Here's how you can help:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Please ensure your PR adheres to:
- Kubernetes coding conventions
- Secure handling of sensitive data
- Comprehensive test coverage
- Clear documentation updates

**NOTE:** Run `make help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

1. Build the installer for the image built and published in the registry:

```sh
make build-installer IMG=<some-registry>/kubeowl:tag
```

**NOTE:** The makefile target mentioned above generates an 'install.yaml'
file in the dist directory. This file contains all the resources built
with Kustomize, which are necessary to install this project without its
dependencies.

2. Using the installer

Users can just run 'kubectl apply -f <URL for YAML BUNDLE>' to install
the project, i.e.:

```sh
kubectl apply -f https://raw.githubusercontent.com/<org>/kubeowl/<tag or branch>/dist/install.yaml
```

### By providing a Helm Chart

1. Build the chart using the optional helm plugin

```sh
kubebuilder edit --plugins=helm/v1-alpha
```

2. See that a chart was generated under 'dist/chart', and users
can obtain this solution from there.

**NOTE:** If you change the project, you need to update the Helm Chart
using the same command above to sync the latest changes. Furthermore,
if you create webhooks, you need to use the above command with
the '--force' flag and manually ensure that any custom configuration
previously added to 'dist/chart/values.yaml' or 'dist/chart/manager/manager.yaml'
is manually re-applied afterwards.

## Contributing

We welcome contributions to KubeOwl! Here's how you can help:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

Please ensure your PR adheres to:
- Kubernetes coding conventions
- Includes appropriate tests
- Updates documentation as needed

**NOTE:** Run `make help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

## License

Copyright 2025.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

