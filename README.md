# Kubernetes
Learning Kubernetes

Pre-Requirement
1.	Install KubeCtl
2.	Install aws-cli
3.	Install aws-azure-login
4.	Install aws-iam-authenticator

run aws-azure-login

Connect to the cluster and view the namespaces:

kubectl get namespaces

NAME STATUS  AGE

abc  ACTIVE  29d

List all pods:

kubectl get pods -n abc

watch kubectl get pods -n abc

Describe Pod:

Any item in kubernetes can be described, simply replace pod with its name

kubectl describe pod pod123 -n abc

A very useful tool to use is kubetail which allows you to tail Kubernetes logs from multiple pods at the same time

Log into EXISTING pod:

Command (sh) must be present in container

kubectl exec -it pod123 -n recast -- bash

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/

k9s - https://github.com/derailed/k9s

K9s provides a curses based terminal UI to interact with your Kubernetes clusters. 

The aim of this project is to make it easier to navigate, observe and manage your applications in the wild. 

K9s continually watches Kubernetes for changes and offers subsequent commands to interact with observed Kubernetes resources.
