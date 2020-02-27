# Deploy Spring Docker Using Helm Chart

**[Quickstart Guide](https://helm.sh/docs/intro/quickstart/)**

### Pre-requisites

The following prerequisites are required for a successful and properly secured use of Helm.

- A Kubernetes cluster
- Deciding what security configurations to apply to your installation, if any
- Installing and configuring Helm.

#### Install Kubernetes or have access to a cluster

- You must have Kubernetes installed. For the latest release of Helm, we recommend the latest stable release of Kubernetes, which in most cases is the second-latest minor release.
- You should also have a local configured copy of `kubectl`.

### Install Helm

From Homebrew (Mac):

```shell
brew install helm
```

From Chocolatey (Windows):

```shell
choco install kubernetes-helm
```

### (Legacy Helm v2) Initialize Helm & Install Tiller

**Helm v3: This Step should NOT be performed**

{
Once you have Helm ready, you can initialize the local CLI and also install Tiller into your Kubernetes cluster in one step:

```shell
helm init --history-max 200
```

TIP: Setting --history-max on helm init is recommended as configmaps and other objects in helm history can grow large in number if not purged by max limit. Without a max history set the history is kept indefinitely, leaving a large number of records for helm and tiller to maintain.

This will install Tiller into the Kubernetes cluster you saw with `kubectl config current-context`.

- TIP: Want to install into a different cluster? Use the `--kube-context flag`.
- TIP: When you want to upgrade Tiller, just run `helm init --upgrade`.
}

### Initialize a Helm Chart Repo

```shell
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

### Install an Example Chart

```shell
helm repo update  # Make sure we get the latest list of charts

# Install a chart from repo
helm install stable/mysql --generate-name
# Output: Released smiling-penguin

# show you a list of all deployed releases
helm ls

# Uninstall the release
helm uninstall smiling-penguin
# Output: Removed smiling-penguin
```

### Three Big Concepts

- A Chart is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster.

- A Repository is the place where charts can be collected and shared. It’s like Perl’s CPAN archive or the Fedora Package Database, but for Kubernetes packages.

- A Release is an instance of a chart running in a Kubernetes cluster. One chart can often be installed many times into the same cluster. And each time it is installed, a new release is created. Consider a MySQL chart. If you want two databases running in your cluster, you can install that chart twice. Each one will have its own release, which will in turn have its own release name.

### Create Your Own Charts

```shell
helm create spring-docker
```

**Edit values.yaml**

```shell
vim spring-docker/values.yaml
```

```yaml
image:
  repository: michaeldqin/spring-docker
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 8080

ingress:
  enabled: true
  annotations:
    {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: myhost-name
      paths: ["/"]
  tls: []
```

**Edit Chart.yaml**

```yaml
apiVersion: v2
name: spring-docker
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application.
appVersion: 1.0.0
```

**Edit templates/deployment.yaml**

Edit the `containerPort` to `8080`

```yaml
ports:
  - name: http
    containerPort: 8080
    protocol: TCP
```

### Quick Install the Helm Chart

```shell
helm install --name my-spring-docker ./spring-docker --set service.type=NodePort
```

### Other Commands

```shell
helm upgrade --name my-spring-docker ./spring-docker
helm rollback --name my-spring-docker ./spring-docker
helm uninstall --name my-spring-docker
```

### Package Charts

```shell
helm package spring-docker
```

### Install a Package

```shell
helm install mydocker ./spring-docker-0.1.0.tgz
```

### Uninstall a Release

```shell
helm uninstall mydocker
```

## Create a Requirement Example

```shell
helm create mychart

nano mychart/requirements.yaml
```

**requirements.yaml**
```yaml
dpendencies:
  - name: mariadb
    version: 0.6.0
    repository: https://kubernetes-charts.storage.googleapis.com
```

**Deploy & Install**
```shell
# Update dependencies
helm dep update ./mychart

# Verify mariadb dependency added to charts directory
tree mychart/

# Deploy to K8s cluster
helm install --name todo ./mychart --set service.type=NodePort

# Grep
kubectl get pods | grep todo

# Package
helm package todo mychart/

# Share and Install by others
helm install --name todo mychart-0.1.0.tgz --set service.type=NodePort

# Add to local repository
helm serve
## Output:
# Regenerating index. This may take a moment.
# Now serving you on 127.0.0.1:8879

# Go to browser
http://127.0.0.1:8879
```

## Helm Chart Repository (JFrog)

[JFrog Helm Chart Repo](https://www.jfrog.com/confluence/display/RTF6X/Helm+Chart+Repositories)

```shell
helm repo add <REPO_KEY> http://<ARTIFACTORY_HOST>:<ARTIFACTORY_PORT>/artifactory/<REPO_KEY> <USERNAME> <PASSWORD>

helm repo add helm-virtual https://jfrog-host:port/artifactory/helm-virtual --username username --password password

# Upload/Deploy spring-docker-0.1.0.tgz to Helm Chart Repo
# Use JFrog UI -> Deploy
https://www.jfrog.com/confluence/display/RTF6X/Deploying+Artifacts

# Update repo
helm repo update

# Install
helm install helm-virtual/spring-docker --generate-name
```
