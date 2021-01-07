# ArgoCD deployment on Kubernetes Cluster
ArgoCD helps to deliver applications to Kubernetes by using the GitOps approach, i.e. when a Git-repository is used as a source of trust, thus all manifest, configs and other data are stored in a repository.

ArgoCD spins up its controller in the cluster and watches for changes in a repository to compare it with resources deployed in the cluster, synchronizing their states.

### ArgoCD Architectural Overview
![Argocd architectrue](https://raw.githubusercontent.com/argoproj/argo-cd/master/docs/assets/argocd_architecture.png "Argocd Components")

### Components of ArgoCD 
ArgoCD consists of the three main components — API server, Repository Server, and Application Controller.
 - API server (pod: argocd-server): controls the whole ArgoCD instance, all its operations, authentification, and secrets access which are stored as Kubernetes Secrets, etc
The API server is a gRPC/REST server which exposes the API consumed by the Web UI, CLI, and CI/CD systems. It has the following responsibilities:

    * application management and status reporting
    * invoking of application operations (e.g. sync, rollback, user-defined actions)
    * repository and cluster credential management (stored as K8s secrets)
    * authentication and auth delegation to external identity providers
    * RBAC enforcement
    * listener/forwarder for Git webhook events

- Application Controller (pod: argocd-application-controller): used to monitor applications in a Kubernetes cluster to make them the same as they are described in a   repository, and controls PreSync, Sync, PostSync hooks .The application controller is a Kubernetes controller which continuously monitors running applications and compares the current, live state against the desired target state (as specified in the repo). It detects `OutOfSync` application state and optionally takes corrective action. It
is responsible for invoking any user-defined hooks for lifecycle events (PreSync, Sync, PostSync)

- Repository Server (pod: argocd-repo-server): stores and synchronizes data from configured Git-repositories and generates Kubernetes manifests .The repository server is an internal service which maintains a local cache of the Git repository holding the application manifests. It is responsible for generating and returning the Kubernetes
manifests when provided the following inputs:

    * repository URL
    * revision (commit, tag, branch)
    * application path
    * template specific settings: parameters, ksonnet environments, helm values.yaml
    	
### Running ArgoCD in Kubernetes
We will install ArgoCD, first create the "argocd" namespace and then we will apply the 1.7.8 manifests (please stick to this argocd namespace, other name will create problems when using manifests directly and not kustomize):

```
$ kubectl create namespace argocd
```

```
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v1.7.8/manifests/install.yaml
```
Next lets check that all the pods are up and running and once they are we can try to connect to the ArgoCD UI
```
$ kubectl get all -n argocd 
NAME                                      READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0       1/1     Running   0          158m
pod/argocd-dex-server-86dc95dfc5-vxsbg    1/1     Running   0          158m
pod/argocd-redis-6fb68d9df5-phttm         1/1     Running   0          158m
pod/argocd-repo-server-5fb8df558f-6rw87   1/1     Running   0          158m
pod/argocd-server-547d9bb879-j2f74        1/1     Running   0          158m

NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/argocd-dex-server       ClusterIP   10.109.65.162    <none>        5556/TCP,5557/TCP,5558/TCP   158m
service/argocd-metrics          ClusterIP   10.102.119.243   <none>        8082/TCP                     158m
service/argocd-redis            ClusterIP   10.108.158.11    <none>        6379/TCP                     158m
service/argocd-repo-server      ClusterIP   10.105.235.242   <none>        8081/TCP,8084/TCP            158m
service/argocd-server           NodePort    10.99.81.251     <none>        80:32715/TCP,443:31491/TCP   158m
service/argocd-server-metrics   ClusterIP   10.98.231.214    <none>        8083/TCP                     158m

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-dex-server    1/1     1            1           158m
deployment.apps/argocd-redis         1/1     1            1           158m
deployment.apps/argocd-repo-server   1/1     1            1           158m
deployment.apps/argocd-server        1/1     1            1           158m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-dex-server-86dc95dfc5    1         1         1       158m
replicaset.apps/argocd-redis-6fb68d9df5         1         1         1       158m
replicaset.apps/argocd-repo-server-5fb8df558f   1         1         1       158m
replicaset.apps/argocd-server-547d9bb879        1         1         1       158m

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     158m
``` 
Since its a NodePort , we can use Server/Worker IP to connect to argocd Server

```
$ kubectl get nodes -o wide
NAME         STATUS   ROLES    AGE    VERSION   INTERNAL-IP  
kub-app001   Ready    master   103d   v1.19.2   10.164.225.6
kub-app002   Ready    master   103d   v1.19.2   10.164.225.8    
kub-app003   Ready    <none>   103d   v1.19.2   10.164.225.7    
kub-app004   Ready    <none>   103d   v1.19.2   10.164.225.10
kub-app005   Ready    <none>   103d   v1.19.2   10.164.225.9
```
Open the browser on kub-app001:8083 and if there are any alerts on the certificate it should be ok because it is a self generated one. On the username put "admin", while the password you can get by running this command (it is the name of the server pod):

```
$ kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d’/’ -f 2
```
Congratulations, now you have successfully installed ArgoCD on your Kubernetes Cluster 

#### Use following links to create your 1st prject using ArgoCD 

- https://levelup.gitconnected.com/integrating-argo-cd-for-your-kubernetes-project-ba6e49dfebaa
- https://www.velotio.com/engineering-blog/gitops-for-kubernetes-using-argo-cd

#### Deploy application using CLI
- https://itnext.io/argocd-setup-external-clusters-by-name-d3d58a53acb0

#### Argocd Spec example & others
- https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/project.yaml
- https://github.com/argoproj/argo-cd/tree/master/docs/operator-manual 
