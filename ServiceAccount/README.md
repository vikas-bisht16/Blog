# <ins> Service Account </ins>

Definition

- Created in a namespace
- Used to allow a process in a pod access to the API Server.
- Default service account = default (no access to the api server)
- Create your own service account.
    - Use it in a RoleBinding or ClusterRoleBinding
    - Use the service account secret to obtain the authentication token & CA certificate.

<br>
<br>

## What we will be covering today
- Creating a pod in default SA
- Will create a SA 
- Creating a deployment which will be using appsa Service Account.

<br>

## **STEP1**
Creating a pod with a defalut service account, as we are not mentioning any SA here.

```
kubectl run -it --rm alpine --image=alpine -- sh
```

So, whenever we create with a SA(Service Account) then its certs & token automatically gets mounted on the pod as well, the location of those credentials are,

```
# cd /var/run/secrets/kubernetes.io/serviceaccount
# ls
# ca.crt     namespace  token
```

Here, we will be using ca.crt & token.

ca.crt - used to make the tls connection with API Server through curl.
token - jwt token, used to authenticate to the cluster.

Through jwt utility you can see the contents of the token,

```
jwt <token>
```

OUTPUT:
```
To verify on jwt.io:

https://jwt.io/#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjI4NWUzZjI2MzdhNDkzMzYwNDJhYzE5MWZiYzFhOGE3OTdkMGNmNmEifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjIl0sImV4cCI6MTY3NTIzNTQ2MywiaWF0IjoxNjQzNjk5NDYzLCJpc3MiOiJodHRwczovL29pZGMuZWtzLnVzLWVhc3QtMi5hbWF6b25hd3MuY29tL2lkLzJGMzMxQzgyNzU1RUYzRTQzNzMxQzhENkUyOTlGQUVGIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJhbHBpbmUiLCJ1aWQiOiIwMDY3NjU4Ni1iODdkLTQ4OWEtYjlhZC0zZWFjMDBhODIzMDEifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlZmF1bHQiLCJ1aWQiOiIzNzNkMDgwZC0wODM1LTQyZDktODQ1NS02OWY4ODc3NTU2MzIifSwid2FybmFmdGVyIjoxNjQzNzAzMDcwfSwibmJmIjoxNjQzNjk5NDYzLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkZWZhdWx0In0.nSmOHEhF3Rl1IeQPZKvAwSBY7JMVQwICnHF_DdPplOjex3jA9Ok1VLgnnukhimVLH8vc_y3W_J3c5IgHk2SxGSGHEhlMeWecnoD87KupkH7sbaq3NbMBNnFPjKhzvOC5smbWrZcsTA3wghnrU6sZFw-kvxivVwQB6I-9j1gTzCLE6yskFLWEM30GhrEvPN7ra26t31iEMbSW0dcP78XP09GD7_WKzdATwjUrLFXSZYig-ZiS4PPQOVA4yFvQJOzO2ebnQ9m-1kr5reMiFfAyQttVQORLU4YS-h3LDH8Li61I3UxaUD8_9togo62ALawCzX1jndaJXH3sDBem8mBrJg

✻ Header
{
  "alg": "RS256",
  "kid": "285e3f2637a49336042ac191fbc1a8a797d0cf6a"
}

✻ Payload
{
  "aud": [
    "https://kubernetes.default.svc"
  ],
  "exp": 1675235463,
  "iat": 1643699463,
  "iss": "https://oidc.eks.us-east-2.amazonaws.com/id/2F331C82755EF3E43731C8D6E299FAEF",
  "kubernetes.io": {
    "namespace": "default",
    "pod": {
      "name": "alpine",
      "uid": "00676586-b87d-489a-b9ad-3eac00a82301"
    },
    "serviceaccount": {
      "name": "default",
      "uid": "373d080d-0835-42d9-8455-69f887755632"
    },
    "warnafter": 1643703070
  },
  "nbf": 1643699463,
  "sub": "system:serviceaccount:default:default"
}
   Issued At: 1643699463 2/1/2022, 12:41:03 PM
   Not Before: 1643699463 2/1/2022, 12:41:03 PM
   Expiration Time: 1675235463 2/1/2023, 12:41:03 PM

✻ Signature nSmOHEhF3Rl1IeQPZKvAwSBY7JMVQwICnHF_DdPplOjex3jA9Ok1VLgnnukhimVLH8vc_y3W_J3c5IgHk2SxGSGHEhlMeWecnoD87KupkH7sbaq3NbMBNnFPjKhzvOC5smbWrZcsTA3wghnrU6sZFw-kvxivVwQB6I-9j1gTzCLE6yskFLWEM30GhrEvPN7ra26t31iEMbSW0dcP78XP09GD7_WKzdATwjUrLFXSZYig-ZiS4PPQOVA4yFvQJOzO2ebnQ9m-1kr5reMiFfAyQttVQORLU4YS-h3LDH8Li61I3UxaUD8_9togo62ALawCzX1jndaJXH3sDBem8mBrJg
```
<br>
<br>

## **STEP2**

Creating the SA,first you can check the manifest from the below command.
```
kubectl create serviceaccount appsa --dry-run=client -o yaml

OUTPUT:

apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: null
  name: appsa
```

Finally, create the SA

```
kubectl create serviceaccount appsa
```
Checking SA,
```
kubectl get sa

OUTPUT:

NAME      SECRETS   AGE
appsa     1         2s
default   1         18d
```

As i have told earlier that whenever we create SA, we are also provided with a secret attached to it, to get that

```
kubectl get secret

OUTPUT:

NAME                  TYPE                                  DATA   AGE
appsa-token-fzmbd     kubernetes.io/service-account-token   3      105s
default-token-st8t8   kubernetes.io/service-account-token   3      18d
```

To get the token, you can use the below command.
```
kubectl get secret appsa-token-fzmbd -o yaml
```
OUTPUT:
```
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1ERXhNekE0TkRReE1Wb1hEVE15TURFeE1UQTRORFF4TVZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBUGozCnN4NXNGNGVUd0dBS25xaW1PVzJKNTlkTmhMUHpaTDM2Wk1DNTZlZENCY0huSVlQanBkRGRmVTFyWlZ0YlJSNEcKeU4vd0NjNG9yZEZ0YmVEdzdReC9Tc3NCZnFoWDE2dDFlYWZhcmpnS1VLUWRkR0ROOS9jOVJRaW1IMm51Q2M0MQpreklKV1pQZTk4Y1Z3MjBlemxBd083S3VIM0ZrbWh5emlsMzloczY5ZWxrVi9EdktDdGRpam5SWUJwMXRERHRmCnhVMC9sQXFUNVFTTUZ1WXJmQkp3c2VyV3lRMXJMOGE3bTkrSDBrR0FwOWdvVE5PYjNpbFRURXJPRzZuTHVUSngKcFlSMWZ4OGE0MzRzb0ZPcjhNOERaSkpETTZLanNFeEl6S3FWK3FCUUZkSHNtOUI3bldNdGVLb1dXS0hTUzVrego4cXVjeXhhdFpNdWNTSFZxSTdVQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZDME8vajh1UmpsbHRJQ3RxZERXeFJ2b0VzVWVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCbHh6UGg0TGdDK1hTTm5CS2lVTzZrOHpGT1pYSUxHYWt5V3orS3hRbjU0eC93UmlyOQpGODdZKzA5Q1Vrb2Zrb2tTUjlwd3YwZXA3UFFNR2hFM1RjU2FGS01nWk01aGo5UmxpQmlqb3RLWGV3K3p0QzdMCnFPUm1vTzljZExxSEFOWGJteEZGZGpac1M4WlJpUGN4LzJQbHU5TFIydjJNVkJFZmtiTVo5QTFiaU1jMndrWlcKMmdvbnlZMUJyNS9YM1FHTDBZNE1PaWc2aXphaUdrZVVLbnNVNGN0L2hpQU5NRS9DTmJzMUgvL0dqcDE1a3ZmbwpXeVlUa2J4UXhHUUZsQ0JJaDZlSHdZdlE1azFwOVBmenZHbTBLb3FxT2FwTTUzd2dtdjd3TEc4ay9kY01tcndmCnY3dm9CSkdMbUZ1TmFXR1cvSTM2ek1PRVZrSzd1M0tTRkFHSgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  namespace: ZGVmYXVsdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkltZHJUR2hSZDBaNlYzbFlTR3BhU1dGVVRqRndia1F0YkVWRFNWVlNNV3BmTldVM2NUZDJkWGR0VXpRaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUprWldaaGRXeDBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbUZ3Y0hOaExYUnZhMlZ1TFdaNmJXSmtJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpYSjJhV05sTFdGalkyOTFiblF1Ym1GdFpTSTZJbUZ3Y0hOaElpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WlhKMmFXTmxMV0ZqWTI5MWJuUXVkV2xrSWpvaU9ERm1abU00WWpZdFlUUTBOQzAwTjJFM0xUaG1aR0l0WXpKaU5HWXpNR1ZsWXpjNUlpd2ljM1ZpSWpvaWMzbHpkR1Z0T25ObGNuWnBZMlZoWTJOdmRXNTBPbVJsWm1GMWJIUTZZWEJ3YzJFaWZRLlZISWRfaHBhYVdRRUhKakZoM3VtV1EyUzBZVGFwa0ZCNVk1SHRqQ08wbXNEZU52SWZfQWZXbkRNdTVmNHBCYWlRV2FES2RZT0I4NjdlZTdNWkZZZkpGQWlzdm5oaDJJRzB2bVBzcU1KNE1aVDdEOXBLLVpsbXNHR2RLYkpPenBlRTVVcFpvSXBZWmp2RG1nQkZRMjJESFBvcmpkZXBSQTF2T1ZEUllxM1ZNRXV0MVNPRXdTdW9abGpFTmNndlh0QWc1X056OEl4bTdaVUNjSFNiekdhYjJJWGNVbTd4RzlFMzhJd2NROVo5UnpGcVllaWZCLTk1Um5WWmh3MjNGYWNBQ2NCd1FycGFVdzFBX3o2b0VYT21PU0YzeXVFMWFlb19hckJyLXJOUDhRVDhwc0FMUDJyUjkwWjh3WHItalpRa3loRjRQbE1JdFFVN2tUTEJLTUZ3dw==
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: appsa
    kubernetes.io/service-account.uid: 81ffc8b6-a444-47a7-8fdb-c2b4f30eec79
  creationTimestamp: "2022-02-01T07:33:45Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:ca.crt: {}
        f:namespace: {}
        f:token: {}
      f:metadata:
        f:annotations:
          .: {}
          f:kubernetes.io/service-account.name: {}
          f:kubernetes.io/service-account.uid: {}
      f:type: {}
    manager: kube-controller-manager
    operation: Update
    time: "2022-02-01T07:33:45Z"
  name: appsa-token-fzmbd
  namespace: default
  resourceVersion: "3526790"
  uid: 5af0be3e-2a34-4411-be9d-276e7138e1e9
type: kubernetes.io/service-account-token
```

Above you can see that we got the ca.crt, namespace & token. As we all know that in k8s tokens are base64 encoded, so the decode that we will be using below command,

```
echo <token> | base64 -d

# -d = decode
```
Now you can use the decoded token to get the information by using jwt, as we did earlier also.

```
jwt <decoded token>
```
OUTPUT:
```
To verify on jwt.io:

https://jwt.io/#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6ImdrTGhRd0Z6V3lYSGpaSWFUTjFwbkQtbEVDSVVSMWpfNWU3cTd2dXdtUzQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImFwcHNhLXRva2VuLWZ6bWJkIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFwcHNhIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiODFmZmM4YjYtYTQ0NC00N2E3LThmZGItYzJiNGYzMGVlYzc5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6YXBwc2EifQ.VHId_hpaaWQEHJjFh3umWQ2S0YTapkFB5Y5HtjCO0msDeNvIf_AfWnDMu5f4pBaiQWaDKdYOB867ee7MZFYfJFAisvnhh2IG0vmPsqMJ4MZT7D9pK-ZlmsGGdKbJOzpeE5UpZoIpYZjvDmgBFQ22DHPorjdepRA1vOVDRYq3VMEut1SOEwSuoZljENcgvXtAg5_Nz8Ixm7ZUCcHSbzGab2IXcUm7xG9E38IwcQ9Z9RzFqYeifB-95RnVZhw23FacACcBwQrpaUw1A_z6oEXOmOSF3yuE1aeo_arBr-rNP8QT8psALP2rR90Z8wXr-jZQkyhF4PlMItQU7kTLBKMF

✻ Header
{
  "alg": "RS256",
  "kid": "gkLhQwFzWyXHjZIaTN1pnD-lECIUR1j_5e7q7vuwmS4"
}

✻ Payload
{
  "iss": "kubernetes/serviceaccount",
  "kubernetes.io/serviceaccount/namespace": "default",
  "kubernetes.io/serviceaccount/secret.name": "appsa-token-fzmbd",
  "kubernetes.io/serviceaccount/service-account.name": "appsa",
  "kubernetes.io/serviceaccount/service-account.uid": "81ffc8b6-a444-47a7-8fdb-c2b4f30eec79",
  "sub": "system:serviceaccount:default:appsa"
}

✻ Signature VHId_hpaaWQEHJjFh3umWQ2S0YTapkFB5Y5HtjCO0msDeNvIf_AfWnDMu5f4pBaiQWaDKdYOB867ee7MZFYfJFAisvnhh2IG0vmPsqMJ4MZT7D9pK-ZlmsGGdKbJOzpeE5UpZoIpYZjvDmgBFQ22DHPorjdepRA1vOVDRYq3VMEut1SOEwSuoZljENcgvXtAg5_Nz8Ixm7ZUCcHSbzGab2IXcUm7xG9E38IwcQ9Z9RzFqYeifB-95RnVZhw23FacACcBwQrpaUw1A_z6oEXOmOSF3yuE1aeo_arBr-rNP8QT8psALP2rR90Z8wXr-jZQkyhF4PlMItQU7kTLBKMF
```

<br>
<br>

## **STEP3**

Here, we will be creating a deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sa-deployment
  labels:
    app: sa
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sa
  template:
    metadata:
      labels:
        app: sa
    spec:
      serviceAccountName: appsa
      containers:
        - name: alpine
          image: byrnedo/alpine-curl
          command:
            - "sh"
            - "-c"
            - "sleep 10000"
```

applying it,
```
kubectl apply -f deployment.yaml
```

Checking,
```
kubectl get deploy

OUTPUT:

NAME            READY   UP-TO-DATE   AVAILABLE   AGE
sa-deployment   1/1     1            1           15s
```

Now describe the pod which is created from this deployment.
```
kubectl describe pod sa-deployment-9c5cd85b-mz2lk

OUTPUT:

Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from appsa-token-fzmbd (ro)
```

As you can see that, this pod is automatically mounted with the token of SA appsa.
