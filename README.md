# ngxpvd-repro

repro issue with nginx provider from traefik

## Create cluster

```shell
k3d cluster create ngxpvd --port 19080:80@loadbalancer --port 19443:443@loadbalancer --k3s-arg "--disable=traefik@server:0"
k3d kubeconfig get ngxpvd > .kubeconfig
chmod 600 .kubeconfig
```

## load variables

```shell
source .env
```

## Deploy Traefik

```shell
helm upgrade --install traefik traefik/traefik --create-namespace --namespace traefik --values traefik/values.yaml
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v3.6/docs/content/reference/dynamic-configuration/kubernetes-ingress-nginx-rbac.yml
kubectl apply -f traefik/ingressclasses.yaml
```

## Add Dashboard

```shell
envsubst < ./traefik/dashboard.yaml | kubectl apply -f -
```

## deploy app

```shell
envsubst < whoami/whoami.yaml | kubectl apply -f -
```

## extract router config from traefik

```shell
curl http://hub-dashboard.localdemo.mageekbox.eu:19080/api/rawdata | jq '.routers | with_entries(select(.key|match("^whoami-whoami-[[:graph:]]+")))'
```
