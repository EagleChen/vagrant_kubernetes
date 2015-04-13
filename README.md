# vagrant_kubernetes
vagrant scripts for bringing up a kubernetes cluster with one master and two nodes.

Inline shell scripts are used to install & configurate kubernetes. 

I use CentOS 7 and Kubernetes-0.14.2-0.1. Fedora would probably be OK.

## How to use
```
vagrant up
```

Duang ! You have a running kubernetes cluster!
If not, try restart node vms ( sometimes node fails to bring up docker after provision )
