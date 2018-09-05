---
layout: post
title: How to Install the prometheus-operator into a namespace, (openshift 3.9/3.10)
---

The [Prometheus Operator for Kubernetes](https://github.com/coreos/prometheus-operator) is useful in monitoring cluster statistics, but can also be used to monitor your applications.   Here's some instructions on how to get it working on Openshift 3.9 or 3.10 inside your custom namespace so you can monitor your application.

Links to the files I'm referencing here:
* [bundle.yaml](https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/bundle.yaml)
* [prometheus.yaml](https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/prometheus.yaml)
* [role.yaml](https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/role.yaml)
* [service.yaml](https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/service.yaml)
* [servicemonitor.yaml](https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/servicemonitor.yaml)

For this example, we'll use a Fuse application, and a namespace called "fuse".    If you want to use a namespace with a different name, go through the yaml files specified here and change the `namespace:` references.

```
oc new-project fuse
oc login -u system:admin
oc create -f bundle.yaml
```

First we create our "fuse" namespace, and then we use the [bundle.yaml](https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/bundle.yaml) to create custom resource definitions for the prometheus-operator, and a role, rolebinding, deployment, and service account necessary for deploying the prometheus-operator.

Note that the [bundle.yaml]((https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/bundle.yaml)) is very similar to the one in the prometheus-operator [instructions](https://github.com/coreos/prometheus-operator/blob/master/README.md), but uses role and rolebinding rather than clusterrole and clusterrolebinding, and includes the custom resource definitions.

```
tcunning@tcunning-OSX:~/prometheus$ oc get pods -n fuse
NAME                                  READY     STATUS    RESTARTS   AGE
prometheus-operator-bccfd55f9-sx5pm   1/1       Running   0          48s
```

Then we need to create a role for prometheus, and a Prometheus instance.   See the [role.yaml]((https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/role.yaml)
) and [prometheus.yaml](https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/prometheus.yaml):

```
tcunning@tcunning-OSX:~/prometheus$ oc apply -f role.yaml
serviceaccount "prometheus" created
role.rbac.authorization.k8s.io "prometheus" created
rolebinding.rbac.authorization.k8s.io "prometheus" created

tcunning@tcunning-OSX:~/prometheus$ oc apply -f prometheus.yaml
prometheus.monitoring.coreos.com "prometheus" created
service "prometheus" created
```

I have a [Fuse 7](https://www.redhat.com/en/technologies/jboss-middleware/fuse) spring boot application entitled "foobar" I'm using.   I've deployed that to my "fuse" namespace.  The "foobar" application has a [Prometheus JMX exporter](https://github.com/prometheus/jmx_exporter) attached that exposes metrics on port 9779.   I need to create a service ([service.yaml]((https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/service.yaml))) and a Prometheus Operator service monitor ([servicemonitor.yaml](https://github.com/cunningt/cunningt.github.io/blob/master/prometheus/servicemonitor.yaml):

```
tcunning@tcunning-OSX:~/prometheus$ oc apply -f service.yaml
service "foobar" created

tcunning@tcunning-OSX:~/prometheus$ oc apply -f servicemonitor.yaml
servicemonitor.monitoring.coreos.com "foobar" created
```

Then expose a route : 

```
oc expose svc/prometheus
```

Using the Prometheus route :

![alt text](https://github.com/cunningt/cunningt.github.io/raw/master/prometheus/prom1.png "Prometheus console, choosing Camel metric")
![alt text](https://github.com/cunningt/cunningt.github.io/raw/master/prometheus/prom2.png "Prometheus console, graphing Camel metric")

