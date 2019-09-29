# Building the project with Skaffold and Kaniko

## Prerequisites
- A Kubernetes cluster such as microk8s .
- Download skaffold
- helm
- kubectl
  

### Installing the Keda controller

Use helm to install the Keda Controller

```
helm repo add kedacore https://kedacore.azureedge.net/helm

helm repo update

helm install kedacore/keda-edge `
    --devel `
    --set logLevel=debug `
    --namespace keda `
    --name keda
```    

### Creating a secret for Kaniko use.

Kaniko will pull and push from a Docker registry.
If you already have a docker config, you can easily create the kaniko secret.

Example
`kubectl -n keda-app create secret generic regcred --from-file=/home/your user/.docker/config.json`

### Build and deploy the application 

Examine the `skaffold.yaml` at the root of the project.
Change the image name according to your registry.

```
apiVersion: skaffold/v1beta15
kind: Config
profiles:
- name: techtalksproducer
  build:
    artifacts:
    - image: balchu/techtalkproducer
      context: src
      kaniko:
        dockerfile: Dockerfile-TechTalksAPI
        buildContext:
          localDir: {}
    cluster:
      dockerConfig: 
        secretName: regcred
      namespace: default
    insecureRegistries: #Use this for local registry.  such as microk8s registry.
    - 10.152.183.39:5000
  deploy:
    kubectl:
      manifests:
        - k8s/TechTalksProducer/*.yml

- name: techtalksconsumer
  build:
    artifacts:
    - image: balchu/techtalksconsumer
      context: src
      kaniko:
        dockerfile: Dockerfile-TechTalksMQConsumer
        buildContext:
          localDir: {}
    cluster:
      dockerConfig: 
        secretName: regcred
      namespace: default
    insecureRegistries:
    - 10.152.183.39:5000
  deploy:
    kubectl:
      manifests:
        - k8s/TechTalksConsumer/*.yml
```

After modifying the `skaffold.yaml`, execute the following command.

To build and run the producer:

`skaffold run -p techtalksproducer`

To build and run the consumer:

`skaffold run -p techtalksconsumer`
