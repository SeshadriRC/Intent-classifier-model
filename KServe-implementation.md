# Kserve Demonstration for Iris model

### Install Cert Manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

### Install KServe CRDs

```
kubectl create namespace kserve

helm install kserve-crd oci://ghcr.io/kserve/charts/kserve-crd \
  --version v0.16.0 \
  -n kserve \
  --wait
```

### Install KServe controller

```
helm install kserve oci://ghcr.io/kserve/charts/kserve \
  --version v0.16.0 \
  -n kserve \
  --set kserve.controller.deploymentMode=RawDeployment \
  --wait
```

### Deploy the Intent Classifier model

```
kubectl create namespace ml

cat <<EOF | kubectl apply -n ml -f -
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: sklearn-iris
spec:
  predictor:
    model:
      modelFormat:
        name: sklearn
      storageUri: "gs://kfserving-examples/models/sklearn/1.0/model"
      resources:
        requests:
          cpu: "100m"
          memory: "512Mi"
        limits:
          cpu: "1"
          memory: "1Gi"
EOF

kubectl get inferenceservice sklearn-iris -n ml
```


`kubectl logs kserve-controller-manager-66678b8767-8q4vl --all-containers -n kserve`

<img width="1916" height="630" alt="image" src="https://github.com/user-attachments/assets/860186e5-5d8d-428f-b5db-e3338bff9e42" />


- All got created

<img width="1917" height="541" alt="image" src="https://github.com/user-attachments/assets/19628ff8-3ac5-4e4d-a8de-ac0437692311" />


### Port-forward to access the model

```
kubectl -n ml port-forward svc/<svc-name> 8080:80
```

### Inference the Model

```
curl -s -X POST http://localhost:8080/v1/models/intent-classifier:predict \
  -H "Content-Type: application/json" \
  -d '{"instances":["I want to cancel my subscription"]}' | jq
```


