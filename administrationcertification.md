
* Certified Kubernetes Administrator: https://www.cncf.io/certification/cka/

* Exam Curriculum (Topics): https://github.com/cncf/curriculum

* Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

* Exam Tips: http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD


## Courses
* https://github.com/kodekloudhub/certified-kubernetes-administrator-course


## ETCD
```
curl -L https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz -o etcd-v3.3.11-linux-amd64.tar.gz
tar xzvf etcd-v3.3.11-linux-amd64.tar.gz -o etcd-v3.3.11-linux-amd64.tar.gz

# start key => value DB
./etcd


$ ./etcdctl set key1 value1
value1
$ ./etcdctl get key1
value1


```


```
kubectl -n kube-system get pods
NAME                                     READY   STATUS    RESTARTS   AGE
coredns-78fcd69978-vz5x4                 1/1     Running   0          34s
etcd-multinode-demo                      1/1     Running   0          64s
kindnet-dswmx                            1/1     Running   0          24s
kindnet-jj8fw                            1/1     Running   0          34s
kube-apiserver-multinode-demo            1/1     Running   0          62s
kube-controller-manager-multinode-demo   1/1     Running   0          62s
kube-proxy-dbvq2                         1/1     Running   0          24s
kube-proxy-spqvt                         1/1     Running   0          34s
kube-scheduler-multinode-demo            1/1     Running   0          64s
storage-provisioner                      1/1     Running   0          61s

```

### ETCD commands
```
ETCD - Commands (Optional)
(Optional) Additional information about ETCDCTL Utility

ETCDCTL is the CLI tool used to interact with ETCD.

ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3.  By default its set to use Version 2. Each version has different sets of commands. 

For example ETCDCTL version 2 supports the following commands:

etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set


Whereas the commands are different in version 3

etcdctl snapshot save 
etcdctl endpoint health
etcdctl get
etcdctl put

To set the right version of API set the environment variable ETCDCTL_API command

export ETCDCTL_API=3



When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.



Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

--cacert /etc/kubernetes/pki/etcd/ca.crt     
--cert /etc/kubernetes/pki/etcd/server.crt     
--key /etc/kubernetes/pki/etcd/server.key


So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:



kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"
```


