# odo-in-k8s
Manifest to run Harmony Connect Remote Access (former ODO) in K8S

Converted original instructions to run adanite/odo_connecter:GEO_v3 in Kubernetes as a Pod - Deployment will follow.

Original command:
```
docker run -d \
--cap-add=NET_ADMIN \
--sysctl net.ipv4.ip_forward=1 \
--device /dev/net/tun --restart=always \
--log-opt max-size=1g \
-e Secret=<CONNECTOR-SECRET> \
adanite/odo_connector:eu_v3
```  
Several interesting requirements had to be solved:
* using sysctl
* mapping device
* using linux capabilities
* DNS (quick hack change core-dns config to forward to external servers)
* logging, solved with run-time option ```--log-file-max-size=1024```

```<CONNECTOR_SECRET>``` provided via ConfigMap, not Secret, since there is no decryption function inside the container and secrets are only base64 encoded.

For offline install, with docker runtime, one may pull the image on a different machine, save it, transfer the archive to K8S master node and docker load it.

Since the manifest includes both ConfigMap and Pod, if Secret must be changed simply run ```kubectl replace --force --grace-period=0 -f``` to have both reinitialized (yes, I know that updating CM will update the env var of container, but it was easier and more consistent during testing).
  
Tested with minikube, which requires --extra-config
```
minikube start --driver=docker --network-plugin=cni --cni=calico --extra-config="kubelet.allowed-unsafe-sysctls=kernel.msg*,net.*" --kubernetes-version=v1.22.2
```
