# Kubernetes basics

* Install Kubectl : `brew install kubectl`
* Check Kubectl : `kubectl version --client`
* Install minikube : `brew install minikube`
* Start minikube cluster : `minikube start`
* Deploy app with the image `node-authentication-jwt_app` : `kubectl create deployment nodeAuthApp --image=node-authentication-jwt_app`
* Check deployments : `kubectl get deployment`