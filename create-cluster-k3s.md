# k3s

```bash
# On the cluster machine
iptables -F
yum install bash-completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
curl http://get.k3s.io -o get.k3s.io.sh
chmod +x get.k3s.io.sh
./get.k3s.io.sh


#  On the source machine,  Get the cluster config on an other machine
iptable -F
yum install bash-completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/local/bin

# on the source code server
scp 
export IPNODE=192.168.0.xxx
# Change the IP  https://127.0.0.1:6443 => Open the port 6443 in TCP in your firewall and redirect to the cluster machine
cat k3s.yaml.tmp |sed "s/127.0.0.1/$IPNODE/g" > k3s.yaml
export KUBECONFIG=$PWD/k3s.yaml
kubectl --insecure-skip-tls-verify  get node

yum install -y epel-release
yum install -y jq
curl https://files.techwhale.io/kubeconfig.sh -o kubeconfig.sh
chmod +x ./kubeconfig.sh
./kubeconfig.sh
chmod +x ./kubeconfig.sh

# Get the cluster config
./kubeconfig.sh
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
-> Cluster name: [default]
-> API URL: [https://...:6643]
-> Service token: [...]
-> Cluster CA:
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----





```
