# Despliega tus aplicaciones en Kubernetes con Helm (II)

Repositorio de código del artículo de [enmilocalfunciona.io](https://enmilocalfunciona.io/despliega-tus-aplicaciones-en-kubernetes-con-helm-ii-monocular-chart-museum/)

## Requirements

- [Kubernetes](https://kubernetes.io/)
- [Helm](https://helm.sh/)
- [Ingress enabled](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)

## Run

Note: We use minikube to simplify the example

### Make sure Ingress is running

```sh
# make sure ingress in enabled
minikube addons enable ingress

# make sure is ingress is running -- Note: This can take up to a minute...
kubectl get pods -n kube-system -w | grep ingress
...
nginx-ingress-controller-5d9cf9c69f-h924f   1/1     Running   0          2m
...
```

### Make sure Helm is running

```sh
kubectl get pods -w -n kube-system | grep tiller
...
tiller-deploy-75f6c87b87-zb47t              1/1     Running   0          10m
...
```

## Monocular

### Add monocular helm repo

```sh
helm repo add monocular https://helm.github.io/monocular
```

### Create custom repo list file

```sh
cat > custom-repos.yaml <<EOF
# `schedule` and `successfulJobsHistoryLimit` are optional parameters. They default to `"0 * * * *"` and `3` respectively
sync:
  repos:
    - name: stable
      url: https://kubernetes-charts.storage.googleapis.com
      schedule: "0 * * * *"
      successfulJobsHistoryLimit: 1
    - name: incubator
      url: https://kubernetes-charts-incubator.storage.googleapis.com
      schedule: "*/5 * * * *"
    - name: monocular
      url: https://helm.github.io/monocular
    - name: bitnami
      url: https://charts.bitnami.com
    - name: brigade
      url: https://brigadecore.github.io/charts
    - name: jetstack
      url: https://charts.jetstack.io
    - name: gitlab
      url: https://charts.gitlab.io/
    - name: elastic
      url: https://helm.elastic.co
      schedule: "*/5 * * * *"
EOF
```

## Install Monocular

```sh
# it may take a couple of minutes
helm upgrade --install monocular -f custom-repos.yaml monocular/monocular --namespace monocular
```

### Add host entry

```sh
echo "$(minikube ip) monocular.local" | sudo tee -a /etc/hosts
```

## Chart Museum

### Install

```sh
helm upgrade --install chart-museum stable/chartmuseum --namespace chart-museum
```

### Check is running

```sh
kubectl get pods -w -n chart-museum
```

### Retrieve Chart Museum service name

```sh
kubectl get services -n chart-museum
```

### Add Chart Museum repo

```sh
# edit custom-repos.yaml and add
sync:
  repos:
    ...
    - name: chart-museum
      url: http://chart-museum-chartmuseum.chart-museum:8080
```

### Deploy new monocular release with repo changes

```sh
helm upgrade --install monocular -f custom-repos.yaml monocular/monocular --namespace monocular
```

## More info

- [Previous post](https://enmilocalfunciona.io/despliega-tu-aplicaciones-en-kubernetes-con-helm/)
