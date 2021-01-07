# ArgoCD deployment on Kubernetes Cluster
ArgoCD helps to deliver applications to Kubernetes by using the GitOps approach, i.e. when a Git-repository is used as a source of trust, thus all manifest, configs and other data are stored in a repository.

ArgoCD spins up its controller in the cluster and watches for changes in a repository to compare it with resources deployed in the cluster, synchronizing their states.

### Components of ArgoCD
![Argocd architectrue](https://miro.medium.com/max/857/1*0cQb00oW-XFvp0lCrxrZvg.png "Argocd Components")

ArgoCD consists of the three main components — API server, Repository Server, and Application Controller.
- API server (pod: argocd-server): controls the whole ArgoCD instance, all its operations, authentification, and secrets access which are stored as Kubernetes Secrets, etc
- Application Controller (pod: argocd-application-controller): used to monitor applications in a Kubernetes cluster to make them the same as they are described in a repository, and controls PreSync, Sync, PostSync hooks
- Repository Server (pod: argocd-repo-server): stores and synchronizes data from configured Git-repositories and generates Kubernetes manifests
    	
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
First app with cluster name

On the UI you should see a message like "No applications yet". So lets create one and we will use the cluster name and not the url. We can use the name even when we deploy apps on the local cluster. The app will look like this:

https://levelup.gitconnected.com/integrating-argo-cd-for-your-kubernetes-project-ba6e49dfebaa
https://www.velotio.com/engineering-blog/gitops-for-kubernetes-using-argo-cd

#### Deploy application using CLI
https://itnext.io/argocd-setup-external-clusters-by-name-d3d58a53acb0
