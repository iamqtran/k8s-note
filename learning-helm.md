# Learning Helm
## What is Helm
1. Helm is a package manager for kubernetes. A package is called a **Chart**.
1. A chart.yaml contains a description of a package.
    1. Chart version.
    1. Chart name.
    1. Other things.
1. Values.yaml is provided values for configuration.
## Helm workflow
1. Helm read the chart.
1. It sends values to templates for generating kubernetes manifest files.
1. Th manifest files are sent to kubernetes cluster.
1. Kubernetes cluster create a request resource inside a cluster.
## Installation Helm
1. curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
1. chmod +x get_helm.sh
1. ./get_helm.sh
## Configuration Helm
1. Helm will try to find this information by reading the environment variable $KUBECONFIG
    1.  **$HOME/.kube/config**
## Using Helm
1. Add a chart repository.
    1. **helm repo add** bitnami https://charts.bitnami.com/bitnami
1. Find a chart to install.
    1. **helm search repo** <package_name>(mysql)
    1. **helm search repo content**
1. Install a Helm chart.
    1. the name of the installation
    1. the chart you want to install
    1. helm install --namespace <kubernetes_namespace> mysql-server bitnami/mysql
    1. helm install --namespace <kubernetes_namespace> mysql-server bitnami/mysql --values values.yaml
    1. helm install --namespace <kubernetes_namespace> mysql-server bitnami/mysql --set Username=admin
1. See the list of what is installed.
    1. **helm repo list**
    1. **helm list**
    1. **helm list --all-namespaces**
1. Upgrade your installation.
    1. helm repo update
    1. helm upgrade --namespace <kubernetes_namespace> mysql-server bitnami/mysql 
    1. helm upgrade --namespace <kubernetes_namespace> mysql-server bitnami/mysql --values values.yaml
    1. helm upgrade --namespace <kubernetes_namespace> mysql-server bitnami/mysql --version 8.0.22
1. Delete the installation.
    1. helm uninstall --namespace <kubernetes_namespace> mysql-server
## Templating and Dry Runs
1. **helm template is a tool for rendering Helm charts into YAML**
    1. **helm repo add** bitnami https://charts.bitnami.com/bitnami
    1. **helm search repo** <package_name>(mysql)
    1. **helm template** <package_name>(mysql)
1. **helm template** have the cleaner output yaml files than **--dry-run**. And no need k8s cluster to run 
1. **--dry-run is designed for debugging**
1. Load the entire chart, including its dependencies.
1. Parse the values.
1. Execute the templates, generating YAML.
1. Parse the YAML into Kubernetes objects to verify the data
1. Send it to Kubernetes
1. Command line:
    1. helm install --namespace <kubernetes_namespace> mysql-server bitnami/mysql --dry-run
    1. Helm will dump a trove of information to standard output
## Build a chart
1. Create a chart
    1. helm create alochym
