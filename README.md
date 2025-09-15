[![Tests and linting](https://github.com/eTribes-Connect-GmbH/helm-app/actions/workflows/test.yaml/badge.svg)](https://github.com/eTribes-Connect-GmbH/helm-app/actions/workflows/test.yaml)

# Local testing

Make sure you are in root directory

## Prerequisites

1. Helm
2. Docker
3. KinD
4. kubectl

### Prepare KinD

```
install KinD
kind create cluster --name ct
kind get kubeconfig --name ct --internal > kubeconfig
chmod 600 kubeconfig
```


## Unit tests

```
helm plugin install https://github.com/helm-unittest/helm-unittest.git
helm unittest chart/app
```

## Chart testing (lint)

```
docker run --rm \
  -v ./:/work -w /work \
  quay.io/helmpack/chart-testing:latest \
  sh -lc 'ct lint --charts charts/app'
```

## Chart testing

```
kind get kubeconfig --name ct --internal > kubeconfig
docker run --rm --network kind \
  -v "$PWD":/work -w /work \
  -v "$PWD/kubeconfig":/kubeconfig \
  -e KUBECONFIG=/kubeconfig \
  quay.io/helmpack/chart-testing:latest \
  sh -lc '
    ct install --charts charts/app 
  '
```

## Smoke test

```
helm install app charts/app -n e2e --create-namespace \
  -f charts/app/ci/ci-values.yaml --wait

kubectl -n e2e create job --from=cronjob/app-e2e-run smoke
kubectl -n e2e wait job/smoke --for=condition=complete --timeout=180s
kubectl -n e2e logs job/smoke
```
