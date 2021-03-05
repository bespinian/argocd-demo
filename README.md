# argocd-demo
A simple demo of ArgoCD

## Setup of ArgoCD on a cluster
First we need to set up ArgoCD on the cluster. For this part we follow the [instruction given by ArgoCD](https://argoproj.github.io/argo-cd/getting_started/). After this step we have the ArgoCD controller installed along with the CRDs that ArgoCD defines.

## Setup of a simple application
Next we deploy a simple application with ArgoCD. For this purpose we first create a new namespace for our application:

```
kubectl create ns awesome-gitops
```

Now we are ready to deploy the Application resource which ArgoCD needs to know the details about our application:

```
kubectl -n argocd apply -f ./application-simple.yml
```

If we check the ArgoCD web application, we see that the application is being deployed.

## Make a change to the application

Lets make a change to the application's Deployment resource. We will upgrade to a new version of `awesome-image` and manually set a new title. The new Deployment ressource should look like this

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: awesome-app-deployment
  labels:
    app: awesome-app
spec:
  replicas: 3
  selector:
    matchLabels:
      component: awesome-app
  template:
    metadata:
      labels:
        component: awesome-app
    spec:
      serviceAccountName: awesome-app-sa
      containers:
        - name: awesome-app
          image: bespinian/awesome-image:2.0.0
          imagePullPolicy: Always
          env:
            - name: APP_TITLE
              value: "Something even awesomer!"
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: awesome-app-config
                  key: dbHost
            - name: APP_VERSION
              valueFrom:
                configMapKeyRef:
                  name: awesome-app-config
                  key: appVersion
          ports:
            - containerPort: 8080
````
When we commit and push this change to this repo, we see that ArgoCD applies the new state to the cluster.

