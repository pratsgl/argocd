# ArgoCD deployment on Kubernetes Cluster
ArgoCD helps to deliver applications to Kubernetes by using the GitOps approach, i.e. when a Git-repository is used as a source of trust, thus all manifest, configs and other data are stored in a repository.

ArgoCD spins up its controller in the cluster and watches for changes in a repository to compare it with resources deployed in the cluster, synchronizing their states.

Components

ArgoCD consists of the three main components — API server, Repository Server, and Application Controller.

    API server (pod: argocd-server): controls the whole ArgoCD instance, all its operations, authentification, and secrets access which are stored as Kubernetes Secrets, etc
	
    Repository Server (pod: argocd-repo-server): stores and synchronizes data from configured Git-repositories and generates Kubernetes manifests
    
	Application Controller (pod: argocd-application-controller): used to monitor applications in a Kubernetes cluster to make them the same as they are described in a repository, and controls PreSync, Sync, PostSync hooks
	
Running ArgoCD in Kubernetes
We will install ArgoCD, first create the "argocd" namespace and then we will apply the 1.7.8 manifests (please stick to this argocd namespace, other name will create problems when using manifests directly and not kustomize):

$ kubectl create namespace argocd

$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v1.7.8/manifests/install.yaml

Next lets check that all the pods are up and running and once they are we can try to connect to the ArgoCD UI

#wait for all pods to be running
$ kubectl get pod -n argocd
$ kubectl port-forward svc/argocd-server -n argocd 8083:80


Open the browser on localhost:8083 and if there are any alerts on the certificate it should be ok because it is a self generated one. On the username put "admin", while the password you can get by running this command (it is the name of the server pod):

kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d’/’ -f 2

First app with cluster name

On the UI you should see a message like "No applications yet". So lets create one and we will use the cluster name and not the url. We can use the name even when we deploy apps on the local cluster. The app will look like this:


https://levelup.gitconnected.com/integrating-argo-cd-for-your-kubernetes-project-ba6e49dfebaa
https://www.velotio.com/engineering-blog/gitops-for-kubernetes-using-argo-cd

#### Deploy application using CLI
https://itnext.io/argocd-setup-external-clusters-by-name-d3d58a53acb0
