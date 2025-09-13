# weDevs-DevOps-Tasks

Pre-Install:

```md
1. Install Ubuntu Desktop in ProxMox
2. Insall Docker
```

## Task 1: Cluster Setup and Application Deployment

### Create a Kubernetes cluster with k3s on your local machine.

#### k3s installing

```bash
curl -sfL https://get.k3s.io | sh -
```

output:

```bash
kubectl get nodes
```

```bash
NAME     STATUS   ROLES                  AGE    VERSION
ubuntu-standard-pc-i440fx-piix-1996   Ready    control-plane,master   5m36s   v1.33.4+k3s1
```

### Use FluxCD (GitOps tool) to bootstrap the Flux controller into the cluster, and configure it to sync the cluster state from a GitHub repository.

#### Installing FluxCD

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

##### Install the Flux Operator

```bash
helm install flux-operator oci://ghcr.io/controlplaneio-fluxcd/charts/flux-operator \
  --namespace flux-system \
  --create-namespace
```

outptu:

```bash
root@ubuntu:/home/ubuntu# helm install flux-operator oci://ghcr.io/controlplaneio-fluxcd/charts/flux-operator   --namespace flux-system   --create-namespace
Pulled: ghcr.io/controlplaneio-fluxcd/charts/flux-operator:0.28.0
Digest: sha256:9ee42975ae78d465c7006fd98431f3e2971326f16f79940f747d997aa1984b94PJ
NAME: flux-operator
LAST DEPLOYED: Sat Sep 13 05:07:56 2025
NAMESPACE: flux-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Documentation at https://fluxcd.control-plane.io/operator/
```

##### Connecting github repository

```bash
export GITHUB_TOKEN=<token>
flux bootstrap github \
  --token-auth=false \
  --owner=arn-ob \
  --repository=arn-ob/weDevs-DevOps-Tasks \
  --branch=main \
  --path=workloads \
  --read-write-key=true \
  --personal
```

output:

```bash
► connecting to github.com
► cloning branch "main" from Git repository "https://github.com/arn-ob/weDevs-DevOps-Tasks.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed component manifests to "main" ("8b7e6b712c00f2db1ecfdcfed1cf70f440fb76f0")
► pushing component manifests to "https://github.com/arn-ob/weDevs-DevOps-Tasks.git"
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
✔ public key: ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYTItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBJk0Az+nm56L8xOx2Raq76AnA/1dkvVN3QDgB1lot/JfzEZSOgFku7zw1URpJ3o+XVlUbkMnWgKV6XvarDFcam69GcVsvsGvPWBHTiKxNrX6d5jvx8988vtwgVXkCfr7/w==
✔ configured deploy key "flux-system-main-flux-system-./workloads" for "https://github.com/arn-ob/weDevs-DevOps-Tasks"
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "main" ("e4ed3dd9bd2e56a86d6e69f156f74e0229f0eb4e")
► pushing sync manifests to "https://github.com/arn-ob/weDevs-DevOps-Tasks.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for GitRepository "flux-system/flux-system" to be reconciled
✔ GitRepository reconciled successfully
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```

Deploying nginx to nginx-app namespace, All the deployment add to workloads/nginx folder 

FluxCD reconcile automatically, Output

```bash
root@ubuntu-Standard-PC-i440FX-PIIX-1996:/home/ubuntu# k get all -n nginx-app
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-66fc78d4b8-6mqf4   1/1     Running   0          72s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/nginx-service   ClusterIP   10.43.212.255   <none>        80/TCP    72s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   1/1     1            1           72s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-66fc78d4b8   1         1         1       72s
```

### Set up SOPS with age for secret encryption in FluxCD

#### Install SOPS

```bash
sudo apt-get install -y age
curl -LO https://github.com/getsops/sops/releases/download/v3.10.2/sops-v3.10.2.linux.amd64
sudo mv sops-v3.10.2.linux.amd64 /usr/local/bin/sops
sudo chmod +x /usr/local/bin/sops
```

- check -

Generate an age Key

```bash
age-keygen -o age.agekey
```

Output

```bash
Public key: age1vsh7t60jklcwj86ynj3jyezfc7p3849vsjffmf5m37tnha2sw4asl39csf
```

cat age.agekey

```bash
# created: 2025-09-13T14:04:30+06:00
# public key: age1vsh7t60jklcwj86ynj3jyezfc7p3849vsjffmf5m37tnha2sw4asl39csf
AGE-SECRET-KEY-10X55RQVGKW7AHW4KR5ATJ5X7P6T8UUF38ZUCY49MDJ6DDTE9SQYS0LG5N7
```

Configure SOPS with the Key

Create `/root/.sops.yaml`:

```yaml
creation_rules:
  - age: >-
      age1vsh7t60jklcwj86ynj3jyezfc7p3849vsjffmf5m37tnha2sw4asl39csf
```

Encrypt a Secret with SOPS `secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-secret
  namespace: default
stringData:
  username: admin_mysql
  password: H3llow?Arn
```

Apply CMD `sops -e -i secret.yaml`

Now the secret yaml is like

```yaml
apiVersion: ENC[AES256_GCM,data:w6I=,iv:HYEfP8JT5yi9bxkpBJ5XoTbTsOKMf+HEY/sm/EvoaVE=,tag:coKA58Ct8F2FNSc/+S7lfQ==,type:str]
kind: ENC[AES256_GCM,data:4V4VZIyF,iv:agAL5DmueTYrOSBEHBg82botM/pHoePA5enzKvmp+aQ=,tag:4O/RH/3pSd16sS52ZyXuxg==,type:str]
metadata:
    name: ENC[AES256_GCM,data:OfYIXFwKl55QRq8e+EpG,iv:pFpx65kebxV2R5zN1WSZi/j6ZzN85whMGyzJOLJcpkE=,tag:zkXsFAj9VJySAMkz+IMlZg==,type:str]
    namespace: ENC[AES256_GCM,data:HIg/BX/WMw==,iv:dClfZTPLPPoPgUmxiUMEUQWQLTfpejc/np2RXIvGBWY=,tag:bkqcY7ZZ3x5ajNTMBjfw/g==,type:str]
stringData:
    username: ENC[AES256_GCM,data:p4ECKUrmwG3Y5wM=,iv:++mhZoCUF0Q67vuOi0PATkxLEHfOemGgRT47Z3tiECE=,tag:5mD7BjlWtYK1Bw8txKJFxA==,type:str]
    password: ENC[AES256_GCM,data:iScbKgcmx5AonA==,iv:4fJu9aaX2fi1OsTNwMQAKHQFocnuU2zFB/dDDo5RKb0=,tag:e/q9WTN1s/tqUpH1lpIj+Q==,type:str]
sops:
    age:
        - recipient: age1vsh7t60jklcwj86ynj3jyezfc7p3849vsjffmf5m37tnha2sw4asl39csf
          enc: |
            -----BEGIN AGE ENCRYPTED FILE-----
            YWdlLWVuY3J5cHRpb24ub3JnL3YxCi0+IFgyNTUxOSBKZ2tUZ0tPUWlvdWhzdTR0
            Smd6VUpEalEvN3Riay9NQm0vK2Zwc3psUncwCkFBYWZuckR1a3RUTmFQSmdUMzRu
            a0YvbkxnUStFaVU5OTdyOHdLcjBXQ0UKLS0tIE8wWVMwVjVQOHpxcWFWS2RtRE5Z
            MC9oQTE2RzNnZkZOYUt5Y0JjUTdvd28KgVCFFaLUIWDaH2lzq2tjpmsVZnKqa1ws
            oYHvNh4Q4i3tv8Whk5YivcwZEL8dApBpc/UTFwFhtv2qlUmmF/jetA==
            -----END AGE ENCRYPTED FILE-----
    lastmodified: "2025-09-13T08:14:31Z"
    mac: ENC[AES256_GCM,data:IefGjc+QVO5pQb1AcVgRRBV/rDKGVgzQYihY4n5XBi1InnkXda40zB+PAhZXpMhbGebiRJvPd0dh+M8ZO1p1yd8L11S/izodFg2Kz51nNcKsGjnC2LdkcZoo9KNCAPYrIdKlG9dNeNtxezpcCE0aeNNmdeR+LK3BdjnVU+f4zEI=,iv:RBRyKJCTn+Eo1ox9+udTiSvFIwfnjNQWMyCOsosRTzQ=,tag:z/S2b6JfBjZQ860QtizivA==,type:str]
    unencrypted_suffix: _unencrypted
    version: 3.10.2
```

Push secret to github, Now Need to sync the private key with FluxCD

```bash
kubectl -n flux-system create secret generic sops-age \
  --from-file=/home/ubuntu/key/age.agekey
```

### Deploy a bare-metal load balancer solution (e.g., MetalLB or OpenELB). (Using MetalLB)

Install MetalLB

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.2/config/manifests/metallb-native.yaml
```

Output

```bash
root@ubuntu-Standard-PC-i440FX-PIIX-1996:/home/ubuntu# k get all -n metallb-system
NAME                              READY   STATUS    RESTARTS      AGE
pod/controller-58fdf44d87-rd72s   1/1     Running   1 (11m ago)   12m
pod/speaker-qjzhz                 1/1     Running   0             12m

NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/metallb-webhook-service   ClusterIP   10.43.107.89   <none>        443/TCP   12m

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   1         1         1       1            1           kubernetes.io/os=linux   12m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           12m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-58fdf44d87   1         1         1       12m
```

Apply metallb ip pool for nginx

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: nginx-ip-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.88.100-192.168.88.200
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: nginx-l2adv
  namespace: metallb-system
spec:
  ipAddressPools:
    - nginx-ip-pool
```

Output

```bash
root@ubuntu-Standard-PC-i440FX-PIIX-1996:/home/ubuntu# k get IPAddressPool -n metallb-system
NAME            AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
nginx-ip-pool   true          false             ["192.168.88.100-192.168.88.200"]
```

```bash
NAME          IPADDRESSPOOLS      IPADDRESSPOOL SELECTORS   INTERFACES
nginx-l2adv   ["nginx-ip-pool"]             
```



### Deploy ingress-nginx using Helm (chart version 4.11.7) via FluxCD

All The yaml are in workloads/ingress-nginx

After Deploying, Output

```bash
root@ubuntu-Standard-PC-i440FX-PIIX-1996:/home/ubuntu# k get ns
NAME              STATUS   AGE
default           Active   133m
flux-system       Active   124m
ingress-nginx     Active   7m53s
kube-node-lease   Active   133m
kube-public       Active   133m
kube-system       Active   133m
metallb-system    Active   33m
nginx-app         Active   80m
```

View all the services of `ingress-nginx`

```bash
NAME                                            READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-7cf74bc745-g2qf2   1/1     Running   0          3m9s

NAME                                         TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.43.72.109    192.168.88.150   80:30137/TCP,443:30441/TCP   3m10s
service/ingress-nginx-controller-admission   ClusterIP      10.43.171.234   <none>           443/TCP                      3m10s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           3m10s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-7cf74bc745   1         1         1       3m9s
```

If i curl the ip, return the 404 Not Found page, Means it works 

```bash
root@ubuntu-Standard-PC-i440FX-PIIX-1996:/home/ubuntu# curl 192.168.88.150
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

### Deploy the following workloads via FluxCD

All The yaml are in workloads/wordpress

