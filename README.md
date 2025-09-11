# Testing

helm plugin install https://github.com/helm-unittest/helm-unittest.git
helm unittest chart/app

docker run --rm \
  -v ./:/work -w /work \
  quay.io/helmpack/chart-testing:latest \
  sh -lc 'git config --global --add safe.directory /work && ct lint --charts charts/app'

