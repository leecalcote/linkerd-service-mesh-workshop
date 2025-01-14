# Lab 2 - Deploy sample application
To play with Linkerd and demonstrate some of it's capabilities, you will deploy the example EmojiVoto application, which is included the Linkerd package.

## What is the EmojiVoto Application?

A microservice application that allows users to vote for their favorite emoji, and tracks votes received on a leaderboard. May the best emoji win.

The end-to-end architecture of the application is shown [here](hhttps://github.com/BuoyantIO/emojivoto/blob/main/assets/emojivoto-topology.png).

It’s worth noting that these services have no dependencies on Linkerd, but make an interesting service mesh example, particularly because of the multitude of services, languages and versions for the reviews service.

Sidecars proxy can be either manually or automatically injected into your pods.

The injector is an admission controller, which receives a webhook request every time a pod is created. This injector inspects resources for a Linkerd-specific annotation (linkerd.io/inject: enabled). When that annotation exists, the injector mutates the pod's specification and adds both an initContainer as well as a sidecar containing the proxy itself.

As part of Linkerd deployment in [Lab 1](../lab-1/README.md), we have deployed the sidecar injector.

### <a name="auto"></a> Deploying Sample App

Linkerd, deployed as part of this workshop, will also deploy the sidecar injector. Let us now verify sidecar injector deployment.


```sh
kubectl get deployment linkerd-proxy-injector -n linkerd
```
Output:

```sh
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
linkerd-proxy-injector   1/1     1            1           9m49s
```

```sh
kubectl get namespace
```

Output:
```sh
NAME           STATUS    AGE       ISTIO-INJECTION
NAME              STATUS   AGE   LINKERD
default           Active   59d
docker            Active   59d
kube-node-lease   Active   59d
kube-public       Active   59d
kube-system       Active   59d
linkerd           Active   10m
```

Now in Meshery in the browser, navigate to the Linkerd adapter's management page from the left nav menu.

On the Linkerd adapter's management page, please enter `default` in the `Namespace` field.
Then, click the (+) icon on the `Sample Application` card and select `Emojivoto Application` from the list.

This will do 3 things: 
1. Deploy `Emojivoto Application` in Emojivoto namespace.
1. Deploys all the Emojivoto services and replica's in the required namespace
1. Injects Linkerd into each component of the `Emojivoto Application`

<small>Manual step for can be found [here](#appendix)</small>

### <a name="verify"></a> Verify Emojivoto deployment

1. Verify that previous deployments are all in a state of AVAILABLE before continuing. **Do not proceed until they are up and running.**

    ```sh
    watch -n emojivoto kubectl get deployment
    ```

2. Inspect the details of the pods

    Let us look at the details of the pods:
    ```sh
    watch -n emojivoto kubectl get po
    ```

    Let us look at the details of the services:
    ```sh
    watch -n emojivoto kubectl get svc
    ```

    Now let us pick a service, for instance productpage service, and view it's sidecar configuration:
    ```sh
    kubectl -n emojivoto get po

    kubectl -n emojivoto describe service svc/web-svc
    ```
#### <a name="linkerd_inject"></a> Inject Linkerd into the sample application

 The emojivoto application is a standalone Kubernetes application that uses a mix of gRPC and HTTP calls to allow the users to vote on their favorite emojis, which means the application can run standalone without support from linkerd service mesh.
 Now we will be injecting linkerd into our sample application
 ```sh
 kubectl get -n emojivoto deploy -o yaml \
  | linkerd inject - \
  | kubectl apply -f -
```

This command retrieves all of the deployments running in the emojivoto namespace, runs the manifest through linkerd inject, and then reapplies it to the cluster. The linkerd inject command adds annotations to the pod spec instructing Linkerd to add (“inject”) the proxy as a container to the pod spec.

You've now added Linkerd to existing services! Just as with the control plane, it is possible to verify that everything worked the way it should with the data plane. To do this check, run:

```sh
linkerd -n emojivoto check --proxy
```

## [Continue to Lab 3 - Access Linkerd via Linkerd Web](../lab-3/README.md)

<hr />
Alternative, manual installation steps below. No need to execute, if you have performed the steps above.
<hr />

## <a name="appendix"></a> Appendix - Alternative Manual Steps

### Deploy emojivoto application
Install emojivoto into the emojivoto namespace by running:
```sh
curl -sL https://run.linkerd.io/emojivoto.yml \
  | kubectl apply -f -
  ```

Before we mesh it, let's take a look at the app. If you're using Docker Desktop at this point you can visit http://localhost directly. If you're not using Docker Desktop, we'll need to forward the web-svc service. To forward web-svc locally to port 8080, you can run:

```sh
kubectl -n emojivoto port-forward svc/web-svc 8080:80
```

### Deploy Emojivoto
Applying this yaml file included in the Istio package you collected in https://run.linkerd.io/emojivoto.yml will deploy the Book info app in you cluster.


```sh
kubectl apply -f https://run.linkerd.io/emojivoto.yml
```

#### <a name="linkerd_inject"></a> Inject Linkerd into the sample application

 The emojivoto application is a standalone Kubernetes application that uses a mix of gRPC and HTTP calls to allow the users to vote on their favorite emojis, which means the application can run standalone without support from linkerd service mesh.
 Now we will be injecting linkerd into our sample application
 ```sh
 kubectl get -n emojivoto deploy -o yaml \
  | linkerd inject - \
  | kubectl apply -f -
```

This command retrieves all of the deployments running in the emojivoto namespace, runs the manifest through linkerd inject, and then reapplies it to the cluster. The linkerd inject command adds annotations to the pod spec instructing Linkerd to add (“inject”) the proxy as a container to the pod spec.

You've now added Linkerd to existing services! Just as with the control plane, it is possible to verify that everything worked the way it should with the data plane. To do this check, run:

```sh
linkerd -n emojivoto check --proxy
```

