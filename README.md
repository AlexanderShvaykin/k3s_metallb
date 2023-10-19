### Create cluster

```
k3d cluster create k3s-metalllb \
  --image rancher/k3s:latest \
  --api-port 6550 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --servers 1 --agents 3 \
  --k3s-arg "--disable=servicelb@server:*"
```

### Check ips

```
docker network inspect k3d-k3s-metalllb | jq '.[0].IPAM.Config[0]'
```

### Deploy apps deps

```
docker run --rm --name redis-master --network k3d-k3s-metalllb -d redis:6
docker run -rm --name postgres -e POSTGRES_PASSWORD=postgres --network k3d-k3s-metalllb -d postgres:11
```

### Deploy metallb

```
kubectl apply -f metallb-native.yaml
kubectl apply -f ipaddress_pools.yaml
kubectl describe ipaddresspools.metallb.io dev -n metallb-system
kubectl apply -f web-app-demo.yaml
kubectl get svc -n web
```

### Info

- https://vhs.codeberg.page/post/kubernetes-macos-load-balancing-k3s-k3d-metallb/
- https://github.com/chipmk/docker-mac-net-connect
- https://k3d.io/v5.6.0/
- https://computingforgeeks.com/deploy-metallb-load-balancer-on-kubernetes/

TODO: sudo route -v add -net 192.168.112.1 -netmask 255.255.255.255 10.0.75.2
