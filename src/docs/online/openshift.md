# OpenShift

An Hawtio console that eases the discovery and management of _hawtio-enabled_ <sup>[1](#f1)</sup> applications deployed on OpenShift.

## Preparation

Prior to the deployment, depending on the cluster types you need to generate either of the _proxying_ or _serving_ certificates.

| Certificate | Description |
| ------------| ----------- |
| _Proxying_  | Used to secure the communication between Hawtio Online and the Jolokia agents. A client certificate is generated and mounted into the Hawtio Online pod with a secret, to be used for TLS client authentication. |
| _Serving_   | Used to secure the communication between the client and Hawtio Online. |


#### Proxying certificates

For OpenShift, a client certificate must be generated using the [service signing certificate][service-signing-certificate] authority private key.

[service-signing-certificate]: https://docs.openshift.com/container-platform/latest/security/certificates/service-serving-certificate.html

Run the following script to generate and set up a client certificate for Hawtio Online:

```sh
$ ./scripts/generate-proxying.sh
```

or if you have Yarn installed, this will also do the same thing:

```sh
$ yarn gen:proxying
```

#### Serving certificates

For OpenShift, a serving certificate is automatically generated for your Hawtio Online deployment using the [service signing certificate][service-signing-certificate] feature.


## Deployment

Now you can run the following instructions to deploy the Hawtio Online console on your OpenShift cluster.

There are two deployment modes you can choose from: **cluster** and **namespace**.

| Deployment Mode | Descripton |
| --------------- | ---------- |
| Cluster | The Hawtio Online console can discover and connect to _hawtio-enabled_ <sup>[1](#f1)</sup> applications deployed across multiple namespaces / projects. <br> **OpenShift:** Use an OAuth client that requires the `cluster-admin` role to be created. By default, this requires the generation of a client certificate, signed with the [service signing certificate][service-signing-certificate] authority, prior to the deployment. See the [Preparation - OpenShift](#openshift) section for more information. |
| Namespace | This restricts the Hawtio Online console access to a single namespace / project, and as such acts as a single tenant deployment. <br> **OpenShift:** Use a service account as OAuth client, which only requires `admin` role in a project to be created. By default, this requires the generation of a client certificate, signed with the [service signing certificate][service-signing-certificate] authority, prior to the deployment. See the [Preparation - OpenShift](#openshift) section for more information. |



You may want to read how to [get started with the CLI](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html) for more information about the `oc` client tool.

To deploy the Hawtio Online console on OpenShift, follow the steps below.

#### Cluster mode

If you have Yarn installed:

```sh
$ yarn deploy:openshift:cluster
```

otherwise (two commands):

```sh
$ oc apply -k deploy/openshift/cluster/
$ ./deploy/openshift/cluster/oauthclient.sh
```

#### Namespace mode

If you have Yarn installed:

```sh
$ yarn deploy:openshift:namespace
```

otherwise:

```sh
$ oc apply -k deploy/openshift/namespace/
```

You can obtain the status of your deployment, by running:

```sh
$ oc status
In project hawtio on server https://192.168.64.12:8443

https://hawtio-online-hawtio.192.168.64.12.nip.io (reencrypt) (svc/hawtio-online)
  deployment/hawtio-online deploys hawtio/online:latest
    deployment #1 deployed 2 minutes ago - 1 pod
```

Open the route URL displayed above from your Web browser to access the Hawtio Online console.


## Authentication

Hawtio Online currently supports two authentication modes: `oauth`, which is configured through `HAWTIO_ONLINE_AUTH` environment variable on Deployment.

| Mode | Description |
| ---- | ----------- |
| oauth | Authenticates requests through [OpenShift OAuth server](https://docs.openshift.com/container-platform/4.9/authentication/understanding-authentication.html). It is available only on OpenShift. |

<a name="f1">1</a>. Containers with a configured port named `jolokia` and that exposes the [Jolokia](https://jolokia.org) API.

