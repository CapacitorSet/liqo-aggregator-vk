---
title: Liqo Dashboard
weight: 4
---

LiqoDash is a web-based UI for [Liqo](https://liqo.io).
It allows you to visualize and control all the Liqo components, as well as manage the status of the Liqo system itself.
You can use LiqoDash to get a general overview of your Liqo resources (both local and remote), see the connected and available peers and start new peerings.
Finally, it allows also to create and modify CRDs using either a built-in YAML/JSON editor or a dynamically generated CRD wizard.

LiqoDash also provides information about the state of _your_ cluster and _all_ other clusters you are connected to.

![Dashboard-UI example](../../images/dashboard/dashboard-ui.png)

## Installing LiqoDash
LiqoDash is installed by default during the installation of Liqo and it is bound to the API server of the cluster where Liqo has been installed.

## Accessing LiqoDash

### NodePort
In order to access the dashboard you need to first get the port on which LiqoDash is exposed, which can be done with the following command:
```
kubectl -n liqo describe service liqo-dashboard | grep NodePort
```
Which will output:
```
Type:          NodePort
NodePort:      https  32421/TCP
```
In this case, the dashboard has been exposed to the port ``32421``
Now, you can access LiqoDash using your master node IP and the service port you just found: ``https://<MASTER_IP>:<LIQODASH_PORT>``

**NOTE:** to get your master node IP, you can run ``kubectl get nodes -o wide | grep master``, and take the
``INTERNAL-IP``

### Port-Forward
A simple way to access the dashboard with a simpler URL than the one specified above is to use ``kubectl port-forward``.
```
kubectl port-forward -n liqo service/liqo-dashboard 6443:443
```
To access LiqoDash you can go to:
```
https://localhost:6443
```
**NOTE:** The PORT exposed with the port forward, that in the example is ``6443``, can be any PORT that is not already used.

### Ingress
LiqoDash can also be exposed using an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress).
You must have an [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
for an Ingress resource to work. If you have one, you can specify the desired host name exporting the variable `DASHBOARD_INGRESS` **before**
installing Liqo, for example:
```
export DASHBOARD_INGRESS=example.liqodash.com
```
Once installed, you can see that an Ingress resource is created:
```
kubectl get -n liqo ingress
```
You can access the dashboard through the provided host name.

### Security certificates
For security reasons, LiqoDash runs over HTTPS, which means that a _Certificate_ is needed to establish the TLS connection.
An x509 self-signed Certificate is generated when the dashboard is deployed, and **must be accepted** by your browser in order to establish
the TLS connection.

The first time you access LiqoDash, it is likely that your browser will show you a warning
that you are trying to access a website that uses a self-signed certificate, and thus it is not trusted
by default. Simply accept the risk and continue to access the dashboard.

If you are exposing LiqoDash through an Ingress, you can either use a certificate provided by a public trusted Certificate Authority (like 
[Let's encrypt](https://letsencrypt.org/)) or a self-signed one.

### Differences between access methods
#### When to use Port-Forward
Using `kubectl port-forward` to access the dashboard service is usually meant for testing/debugging purposes. It is not a 
long term solution, as it require to always run the port-forward and keep it active. However, if you just want to access
the dashboard via localhost, this is the easiest method.
#### When to use NodePort
Using a NodePort means that a specific port is opened on all the Nodes, and traffic sent to this port is forwarded
to a service (in this case the dashboard). It is not recommended to use this method to directly expose a service
on the internet, but if you are going to use the dashboard on the same machine your cluster is, that is a simple
and valid solution.
#### When to use Ingress
Using Ingress is probably the best way to expose LiqoDash, especially if you want to access it through the internet. 
It can provide load balancing, SSL termination and name-based hosting, letting you access the dashboard using
a host name instead of just its IP. Because there many types of Ingress controllers, each one with different
capabilities, it may require some work to properly set up.

## Authentication

### Login with Bearer Token
For security, LiqoDash requires a Bearer Token to log in.
This token is stored within a _secret_, which is created by default during the installation, along with a _service account_ and a _cluster role binding_.

To find the token to use to log in, run the following:
```
kubectl -n liqo describe secret $(kubectl -n liqo get secret | grep liqodash | awk '{print $1}')
```
which should print something like this:
```
Name:         liqodash-admin-sa-token-94v8x
Namespace:    liqo
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: liqodash-admin-sa
              kubernetes.io/service-account.uid: ad421b68-7ca5-4f2b-9022-454eb42f880d

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  4 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Img0d3lBRjRGWTlYbndfcnZFeWNQR2VQX3dRVjhfXzBVLTdlTG95Tm9QMW8ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJsaXFvIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImxpcW9kYXNoLWFkbWluLXNhLXRva2VuLTk0djh4Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImxpcW9kYXNoLWFkbWluLXNhIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYWQ0MjFiNjgtN2NhNS00ZjJiLTkwMjItNDU0ZWI0MmY4ODBkIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmxpcW86bGlxb2Rhc2gtYWRtaW4tc2EifQ.ZX4SgxepLjDWMYtvlWUfR3Qjzhf80Jq-17JzF7DSZVMJKvqgah0JIG9Ieqj6DBQr-0xxWnTW6hosNjcdf6pm62SbuiSMwyE3xS_j3dAmCQHx5umGbnTjp6GUaMu8JiFtajOpU7-9f06W5g4I44LF1-3FwgG3OY6vVdL6CypWfjumwgh_yLKE9h7tjKl8CiSfNuLVDWHL4l07W9fEeed8lNmFg4FlvOVHmFglTjz20VKEeu964pNlgK0MRGo_cVnDJyWl7cdeEmR0qfiPup5AMQLWUvlX9RTB7UTiRyw9YYZXPrsX5sdUMVuWb-G9ZQ8eABQI7BAs4uCouuoWmIDzag
```

Now, (1) copy the token, (2) paste it in the login screen, and (3) sign in.

**NOTE:** the LiqoDash Service Account has admin privileges. To find out more about permissions in Kubernetes
read the official [authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
and [authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) documentation.

### Login with OIDC
If you have an OIDC provider available in your cluster (such as Keycloak), you can use it to access the dashboard and bypass the standard login with the token.
In order to bind the dashboard with you OIDC provider, you have to edit the dashboard _configmap_, which represents the structure that contains the Dashboard configuration.
To do so, run the following command:
```
kubectl edit -n liqo configmap liqo-dashboard-configmap
```
You should see a YAML representation of the configmap, such as the following:
```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
...
  name: liqo-dashboard-configmap
  namespace: liqo
  resourceVersion: "012345"
  selfLink: /api/v1/namespaces/liqo/configmaps/liqo-dashboard-configmap
  uid: 8e48f478-993d-11e7-87e0-901b0e532516
data:
  oidc_client_id: ""
  oidc_client_secret: ""
  oidc_provider_url: ""
  oidc_redirect_uri: ""
kind: ConfigMap
```
Change the values under ``data`` and save the file.
Then, restart the dashboard:
```
kubectl rollout restart deployment liqo-dashboard -n liqo
```
From now on, any attempt to login will be redirected to your OIDC provider for the authentication.

**NOTE:** you can always revert to the token login authentication by editing the configmap.

## Metrics Integration
One of the key feature of LiqoDash is to show the real time status of your cluster (in terms of cpu and memory), as well as the other clusters you
are connected to. In order to show the **actual** consumption of resources you need to have a metrics server running in
your cluster. Otherwise, LiqoDash will consider every resource consuming so much cpu and memory as specified in the _request_ parameter of that resource
(if specified).

You can install a metrics server following the official [documentation](https://github.com/kubernetes-sigs/metrics-server#deployment).
The easiest way to check if `metrics-server` is installed and working properly is to run `kubectl top pod` or `kubectl top node`.