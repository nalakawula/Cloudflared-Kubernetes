# Deploy Cloudflared to kubernetes

This repository contain manifest to deploy cloudflared tunnel to kubernetes.
In my case I use K3s.

We need to install cloudflared cli in our host to login, to create tunnel, and to create route.

Login to cloudflare

```bash
cloudflared login
```

Create tunnel called k3s or whatever you want to name it.
Take note on the output, similiar too `/home/User/.cloudflared/{uuid}.json`

```bash
cloudflared tunnel create k3s
```

Copy the json to this directory

```bash
cp /home/User/.cloudflared/{uuid}.json .
```

Replace {uuid} inside `kustomization.yml` with yours.

Update the ingress part in the `config.yml`.
For example i want to route `ghost.kube.my.id`

```yml
tunnel: k3s
credentials-file: /etc/cloudflared/creds/credentials.json
ingress:
- hostname: ghost.kube.my.id
  service: http://ghost-svc.ghost.svc.cluster.local
- service: http_status:404
```

After that let create route

```bash
cloudflared tunnel route dns k3s ghost.my-domain.tld
```

Now we are ready to deploy cloudflared to kubernetes

```bash
kubectl create ns ingress
kubectl kustomize ./
kubectl apply -k ./
```

Check result:

```bash
kubectl -n ingress get all
NAME                               READY   STATUS    RESTARTS   AGE
pod/cloudflared-7b446955cc-jr6k7   1/1     Running   0          19h

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cloudflared   1/1     1            1           26h

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/cloudflared-55d8bc95d4   0         0         0       26h
replicaset.apps/cloudflared-7867545b65   0         0         0       25h
replicaset.apps/cloudflared-7b446955cc   1         1         1       19h
replicaset.apps/cloudflared-7d94587b56   0         0         0       25h
replicaset.apps/cloudflared-7f6ddbff66   0         0         0       19h
replicaset.apps/cloudflared-7ff947b998   0         0         0       25h
```

Check log

```bash
kubectl -n ingress logs -l app=cloudflared -f
2025-01-08T07:34:14Z INF Registered tunnel connection connIndex=0 connection=353d1577-001a-4c98-be08-ad010abb49c1 event=0 ip=198.41.200.233 location=sin14 protocol=http2
2025-01-08T07:34:14Z INF Registered tunnel connection connIndex=1 connection=ce3e3bc9-3124-4eb3-b3e1-5a5db780b15b event=0 ip=198.41.192.67 location=cgk01 protocol=http2
2025-01-08T07:34:15Z INF Registered tunnel connection connIndex=2 connection=202b7c79-2695-4c78-a3c5-a1355aaa4c87 event=0 ip=198.41.200.73 location=sin12 protocol=http2
2025-01-08T07:34:15Z INF Registered tunnel connection connIndex=3 connection=37d9aec8-31ca-4d1f-869a-19fd30ba293b event=0 ip=198.41.192.37 location=cgk01 protocol=http2
```
