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
- Here we are uploading a `.pkl` to the git repo, so first we need to generate the `.pkl` file as it doesn't exit now
- Generate here

<img width="1396" height="237" alt="image" src="https://github.com/user-attachments/assets/c17eb384-8f2a-4348-ac7d-fae9fe550520" />


```bash
py -3.12 -m venv .venv
source .venv/Scripts/activate

py -3.12 -m pip install -r requirements.txt
py -3.12 model/train.py 
```

<img width="1556" height="270" alt="image" src="https://github.com/user-attachments/assets/ad0fc25b-d420-4cec-bb96-22056f1a9235" />

<img width="1267" height="203" alt="image" src="https://github.com/user-attachments/assets/fd703e9b-6335-4edd-a2dd-1fdab4119083" />

## Create a tag

Tag icon --> Create new release

<img width="1902" height="858" alt="image" src="https://github.com/user-attachments/assets/7d3e1b2d-d22b-4793-bb8b-a525cedac8a2" />

- Create a new tag

<img width="1126" height="602" alt="image" src="https://github.com/user-attachments/assets/5a3eee71-f943-4230-97d4-d0d4b2aeb688" />

<img width="1590" height="755" alt="image" src="https://github.com/user-attachments/assets/0437679f-aa4b-44d1-bc38-c8cdf81d1fde" />

<img width="1637" height="375" alt="image" src="https://github.com/user-attachments/assets/2c1f8ced-82c1-4d67-a4ad-fde46f18101f" />

<img width="1507" height="622" alt="image" src="https://github.com/user-attachments/assets/0aac04a5-d2ef-49b8-a664-6635de9acab6" />

<img width="1881" height="790" alt="image" src="https://github.com/user-attachments/assets/a2aedb73-0485-458c-9f79-80e981a2a8c7" />


- use below yaml, replace the storeUri

```yaml
kubectl create namespace intent

cat <<EOF | kubectl apply -n intent -f -
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: intent-classifier
spec:
  predictor:
    model:
      modelFormat:
        name: sklearn
      storageUri: "https://github.com/SeshadriRC/Intent-classifier-model/releases/download/1.0/intent_model.pkl"
      resources:
        requests:
          cpu: "100m"
          memory: "512Mi"
        limits:
          cpu: "1"
          memory: "1Gi"
EOF

kubectl get inferenceservice intent-classifier -n intent
```

<img width="1852" height="525" alt="image" src="https://github.com/user-attachments/assets/0c9d7e49-2749-49ef-8f4b-7e259c62124a" />

- port forward and test it on other terminal

```bash
kubectl port-forward svc/intent-classifier-predictor 8080:80 -n intent --address 0.0.0.0
```

```bash
curl -s -X POST http://localhost:8080/v1/models/intent-classifier:predict \
  -H "Content-Type: application/json" \
  -d '{"instances":["good night"]}' | jq
---

<img width="1347" height="573" alt="image" src="https://github.com/user-attachments/assets/9a968586-852b-4364-947d-ab04ef56836f" />

