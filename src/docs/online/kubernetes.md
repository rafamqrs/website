# Kubernetes

An Hawtio console that eases the discovery and management of _hawtio-enabled_ <sup>[1](#f1)</sup> applications deployed on Kubernetes.

## Preparation

Prior to the deployment, depending on the cluster types you need to generate either of the _proxying_ or _serving_ certificates.

| Certificate | Description |
| ------------| ----------- |
| _Proxying_  | Used to secure the communication between Hawtio Online and the Jolokia agents. A client certificate is generated and mounted into the Hawtio Online pod with a secret, to be used for TLS client authentication. |
| _Serving_   | Used to secure the communication between the client and Hawtio Online. |


#### Proxying certificates

For Kubernetes, proxing certificates are disabled by default and you don't need to go through the steps.

>  :warning: This means that client certificate authentication between Hawtio Online and the Jolokia agents is not available by default for Kubernetes, and the Jolokia agents need to disable client certificate authentication so that Hawtio Online can connect to them. You can still use TLS for securing the communication between them.
>
>  It is possible to use a proxying client certificate for Hawtio Online on Kubernetes; it requires you to generate or provide a custom CA for the certificate and then mount/configure it into the Jolokia agent for its client certificate authentication.

#### Serving certificates

For Kubernetes, a serving certificate must be generated manually. Run the following script to generate and set up a certificate for Hawtio Online:

```sh
$ ./scripts/generate-serving.sh [-k tls.key] [-c tls.crt] [SECRET_NAME] [CN]
```

or:

```sh
$ yarn gen:serving [-k tls.key] [-c tls.crt] [SECRET_NAME] [CN]
```

You can provide an existing TLS key and certificate by passing parameters `-k tls.key` and `-c tls.crt` respectively. Otherwise, a self-signed `tls.key` and `tls.crt` will be generated automatically in the working directory and used for creating the serving certificate secret.

You can optionally pass `SECRET_NAME` and `CN` to customise the secret name and Common Name used in the TLS certificate. The default secret name is `hawtio-online-tls-serving` and CN is `hawtio-online.hawtio.svc`.


Now you can run the following instructions to deploy the Hawtio Online console on your Kubernetes cluster.

You may want to read how to [get started with the CLI](https://kubernetes.io/docs/reference/kubectl/overview/) for more information about the `kubectl` client tool.

To deploy the Hawtio Online console on Kubernetes, follow the steps below.

#### Cluster mode

If you have Yarn installed:

```sh
$ yarn deploy:k8s:cluster
```

otherwise:

```sh
$ kubectl apply -k deploy/k8s/cluster/
```

#### Namespace mode

If you have Yarn installed:

```sh
$ yarn deploy:k8s:namespace
```

otherwise:

```sh
$ kubectl apply -k deploy/k8s/namespace/
```
## Deployment

There are two deployment modes you can choose from: **cluster** and **namespace**.

| Deployment Mode | Descripton |
| --------------- | ---------- |
| Cluster | The Hawtio Online console can discover and connect to _hawtio-enabled_ <sup>[1](#f1)</sup> applications deployed across multiple namespaces / projects. |
| Namespace | This restricts the Hawtio Online console access to a single namespace / project, and as such acts as a single tenant deployment.

## Authentication

Hawtio Online currently supports two authentication modes: `oauth`, which is configured through `HAWTIO_ONLINE_AUTH` environment variable on Deployment.

| Mode | Description |
| ---- | ----------- |
| form | Authenticates requests with [bearer tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) throught the Hawtio login form. |

### Creating user for Form authentication

With the Form authentication mode, any user with a bearer token can be authenticated. See [Authenticating](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) for different ways to provide users with bearer tokens.

Here we illustrate how to create a `ServiceAccount` as a user to log in to the Hawtio console as an example. See [Creating a Hawtio user for Form authentication](docs/create-user.md) for more details.

<a name="f1">1</a>. Containers with a configured port named `jolokia` and that exposes the [Jolokia](https://jolokia.org) API.
