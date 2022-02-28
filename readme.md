# Projeto Kubernets/ArgoCD
    Automatizando o pipeline usando kubernetes e argocd

    Instalei localmente o Kind (para criar meu cluster)

# Criar um cluster kubernets    
    1: kind create cluster --name dataops
    2: kind create cluster --name dataops --config ./kind_config.yaml
# Instalação do ArgoCd

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get all -n argocd
kubectl port-forward service/argocd-server 8080:80 -m argocd

pegando a senha
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
PSwldHZt8R20tpI3

criar uma aplicação no ARGOCD, configurar o git
 kubectl get applications -n argocd


# subindo projeto teste
fazer um fork do projeto git@github.com:KubeDev/k8s-web-color-page.git

kubectl apply -f .\k8s\deployment.yaml
kubectl port-forward svc/webcolor 8080:80
kubectl delete -f .\k8s\deployment.yaml

# automatizando o argocd com application.yaml
kubectl get applications web-color -n argocd -o yaml > application.yaml
kubectl apply -f .\application.yaml

# configurando em ambiente diferente