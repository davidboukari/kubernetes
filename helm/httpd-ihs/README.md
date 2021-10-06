# httpd-ihs

## Install
```
helm registry login -u davbou https://index.docker.io/v1/
Password:
Login Succeeded
```

## Create a secret for the registry
```
kubectl -n dev create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=myuser --docker-password=mypasswd --docker-email=mymail
kubectl get secret regcred --output=yaml^C

kubectl -n dev get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
```

## Create a pod which use this secret
```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred


helm -n dev install httpd-ihs-dijon-fr .  -f values-dijon-fr.yaml
helm -n dev install httpd-ihs-paris-fr .  -f values-paris-fr.yaml
```

## List release
```
helm -n dev list
NAME              	NAMESPACE	REVISION	UPDATED                                 	STATUS  	CHART          	APP VERSION
httpd-ihs-dijon-fr	dev      	2       	2021-10-06 21:33:45.890088431 +0200 CEST	deployed	httpd-ihs-0.1.0	1.16.0
```

## Show a status
```
helm  -n dev status httpd-ihs-dijon-fr
NAME: httpd-ihs-dijon-fr
LAST DEPLOYED: Wed Oct  6 18:22:33 2021
NAMESPACE: dev
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace dev -o jsonpath="{.spec.ports[0].nodePort}" services httpd-ihs-dijon-fr)
  export NODE_IP=$(kubectl get nodes --namespace dev -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT

```

## Upgrade
```
helm -n dev upgrade httpd-ihs-dijon-fr . -f values-dijon-fr.yaml
```
