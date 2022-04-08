
* Certified Kubernetes Administrator: https://www.cncf.io/certification/cka/

* Exam Curriculum (Topics): https://github.com/cncf/curriculum

* Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

* Exam Tips: http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD


## Courses
* https://github.com/kodekloudhub/certified-kubernetes-administrator-course


### ETCD
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
