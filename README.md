# Argo CD Demonstrator

Demonstrate the functionalities of Argo CD.

# Prerequisites
The following tools must be installed on your computer in order to run this demonstration or do the tutorial
- docker
- minikube
- ansible
- helm

# Demonstration
First, let's create a minikube cluster and install Argo CD on it
```shell
$ cd ansible
$ ansible-playbook all.yml
``` 
Then, make sure you can access Argo CD (replace 9080:9443 with the target ports you want to use):
```shell
$ kubectl port-forward svc/argocd-server -n argocd 8443:443
```
Open your browser and go to the url: `localhost:8443` and enter the login/password combination admin/admin
You should see several applications. Two app of apps applications and three applications. Depending on the speed of your computer, it can be that some apps are still syncing. But after several minutes, they should all become green and in have the status synced.

Have a look at the settings.
## Promoting an app to production
Just copy the application yaml from `environments/dev/templates` to `environments/pro/templates`

## Integration testing an application

# Tutorial

Start by creating an empty minikube cluster. Add ingress to it and make sure that kubectl can see this cluster.
```shell
$ minikube start -p argo-cd-tutorial
$ minikube addons enable ingress -p argo-cd-tutorial 
$ minikube update-context -p argo-cd-tutorial
```
Once there is a running cluster, install Argo CD using a helm chart. E.g.:
```shell
$ helm install argo-cd argo/argo-cd -n argo-cd --create-namespace
```
This command probably fails, unless the repository containing the Argo CD helm chart is added to helm. So let's do that and rerun the install:
```shell
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm install argo-cd argo/argo-cd -n argo-cd --create-namespace
```
You should see some instruction in the terminal that allows you to access the web interface.

But... hey... this is cumbersome. I want to provide my own password and ingress, how do I do that?
Start by creating a values.yaml file and add the following section to it:
```yaml
server:
  ingress:
     enable: true
     https: false
     hosts:
       - argocd.minikube.local
  extraArgs:
    - --insecure
```
As certificates is beyond the scope of this tutorial, the ingress is configured without.



# Content of the folders
## ansible
This directory contains four ansible playbooks:
- all.yml
  : creates and configures a minikube cluster and installs Argo CD on it. It does all steps done by the create_cluster and deploy-helm-argocd playbooks.

- create_cluster.yml
  : creates and configures a minikube cluster.

- delete_cluster.yml
  : deletes/removes the created minikube cluster.

- deploy-helm-argocd.yml
  : Deploys the helm chart for Argo CD and configures Argo CD for this demonstration. This includes adding projects, adding the links to the repositories and deploying two root applications. One for development and one for production. In order to demonstrate the app of apps functionality.

## appsOnDev
This directory contains all application manifests that should be deployed in the dev namespace and project. For now, it contains two identical applications, one using helm for deployment and one using kustomize.
## appsOnPro
This directory contains all application manifests that should be deployed in the pro namespace and project. For now, it contains one applications, that uses helm for deployment.
## charts
This directory contains the helm-charts for the applications that can be deployed
## deployments
This directory contains the kustomize definitions for the applications that can be deployed
## iTestEnvs
This directory contains the definition of the integration testing environments that should be deployed.

# faq
## minikube fails because docker data-dir is mounted noexec
- Create a directory on a volume that is executable.
- Add this directory by adding `   "data-root": "<your-dir>",` to `/etc/docker/daemon.json`
- run `sudo systemctl restart docker`

