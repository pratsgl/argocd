
https://levelup.gitconnected.com/integrating-argo-cd-for-your-kubernetes-project-ba6e49dfebaa

https://www.velotio.com/engineering-blog/gitops-for-kubernetes-using-argo-cd

From Master host

export KUBECONFIG=/home/vagrant/.kube/admin.conf

kubectl -n staging --address 172.42.42.100 port-forward svc/message-app  8081:80
Forwarding from 172.42.42.100:8081 -> 8080
Handling connection for 8081




[vagrant@kmaster ~]$ kubectl -n guestbook --address 172.42.42.100 port-forward service/local-server-guestbook-app-helm-guestbook  8081:80
Forwarding from 172.42.42.100:8081 -> 80
Handling connection for 8081

Deploy application using CLI
https://itnext.io/argocd-setup-external-clusters-by-name-d3d58a53acb0
