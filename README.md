# Deploy Spring Docker Using Helm Chart

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

### Initialize a Helm Chart Repo

```shell
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

### Install an Example Chart

```shell
helm repo update  # Make sure we get the latest list of charts

helm install stable/mysql --generate-name
# Released smiling-penguin

# show you a list of all deployed releases
helm ls

# Uninstall the release
helm uninstall smiling-penguin
# Removed smiling-penguin
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
