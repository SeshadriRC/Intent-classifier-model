# Kserve Demonstration for sample model

### Install Cert Manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

### Install KServe CRDs

```
export NAMESPACE="kserve"

helm install kserve-crd oci://ghcr.io/kserve/charts/kserve-crd \
  --version v0.20.0 \
  --namespace $NAMESPACE \
  --create-namespace
```

### Install KServe controller

```
helm install kserve oci://ghcr.io/kserve/charts/kserve-resources \
  --version v0.20.0 \
  --set kserve.controller.deploymentMode=Standard \
  --namespace $NAMESPACE
```

## Apply cluster resources
```
kubectl apply --server-side -f https://github.com/kserve/kserve/releases/download/v0.20.0/kserve-cluster-resources.yaml  
kubectl get clusterservingruntime
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


```
kubectl edit inferenceservice sklearn-iris -n ml  
  
  
modelFormat:
  name: sklearn
  version: "1"
```

<img width="1916" height="630" alt="image" src="https://github.com/user-attachments/assets/860186e5-5d8d-428f-b5db-e3338bff9e42" />


- All got created

<img width="1917" height="541" alt="image" src="https://github.com/user-attachments/assets/19628ff8-3ac5-4e4d-a8de-ac0437692311" />


### Port-forward to access the model

<img width="1077" height="240" alt="image" src="https://github.com/user-attachments/assets/c005adb4-286a-4533-b443-80a51c55bbfa" />

```
kubectl -n ml port-forward svc/<svc-name> 8080:80
```

### Inference the Model

```
curl -s -X POST http://localhost:8080/v1/models/sklearn-iris:predict \
  -H "Content-Type: application/json" \
  -d '{"instances":[[5.9,3.0,5.1,1.8]]}' | jq
```

<img width="1127" height="307" alt="image" src="https://github.com/user-attachments/assets/1286a598-f2da-48c7-a302-8f9a1458274b" />

---
# Kserve Demonstration for Iris model

- Install cert manager , follow above
- Install CRD, Controller and apply cluster resources - follow above steps


---
