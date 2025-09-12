+++
title = "Signoz for dummies: Sending K8s data to Signoz cloud"
weight = 10
+++

Over the last few days, I’ve been diving into Kubernetes, OpenTelemetry, observability, and SigNoz, and documenting my takeaways along the way. <Br>

In this post, we’ll walk through how to use SigNoz to collect Kubernetes data, send it to the SigNoz Cloud, and set up an alert that triggers whenever a pod gets stuck in a Pending state.
<Br>
So lets get started :<Br>
Firstly, 
What is Kubernetes, what is this thing we are doing here, where does Signoz come into the picture, can we do it without Signoz.
Will answer all of them.

Kubernetes?
> Kubernetes — cluster orchestration for containers, with this you can manage and track a lot of containers across different clusters.
Where it's useful?<br>
— Not for everyday use cases and that's why a lot of people don't know about it, but in cases where the resources get huge and you have to manage a lot of cluster, like for large applications.
> 

And what does signoz exactly do?
> It is a full stack thing ( in more formal terms, full stack open source APM ) to monitor how the kubernetes cluster is. Like the state of each pod, container, total memory used, cpu utilisation of each container. It uses the famous OpenTelemetry to collect metrics from your K8s cluster.
> 

Wait, what’s OpenTelemetry?
> Instrumentation standard for application monitoring.
It basically sends the data to your required backend.<Br>
Main feature of OpenTelemetry is its vendor neutral APIs and data formats, and it can act as delivery pipeline for all three of traces, metrics and logs, to later convert to most exciting data formats to another. More about components of OpenTelemetry later: Receivers, collectors, and exporters.
> 

```
Goal of OpenTelemetry can be summarized as:
Open standards based interoperability and vendor neutral observability 
ecosystem.
```

```
Gist :
Kubernetes → lots and lots of different kinds of data. 
And OTel, gives an open source solution to collect and export this data in 
any format. 
And Signoz receveives this data, and makes it prettier for analysis and querying.
Kubernetes, Opentelemtry - are top 2 projects in open source, and are associated 
with modern microservices architecture( nowadays, it’s prevalent, different 
services for different apps), and that’s where Signoz comes in, to monitor
Kubenetes cluster with Opentelemetry, to collect, manage and display your
observability data.

Why is this important?
Helps you understand what’s happening in your cluster, making it easier to 
troubleshoot and making systems are running smoothly.

```


Open Telemetry has 3 components :

1) Receivers : they collect data from all sources and convert them to an internal format for sending. OTLP is OpenTelemetry’s intermediate data format.
2) Processors : for re-processsign the data before finally sending it, and mechanisms like batching it together, and sending again if it fails, retry mechanism.
3) Exporters : they further send the processed data after receiving from processors, by converting them to a target data-sink specific format.

Helm presets like *kubeletMetrics*, *logsCollection*, and *clusterMetrics* make it easy to configure receivers and processors without writing full config blocks manually.

# So let’s get started

We will simulate a small kubernetes cluster on our pc using kind.

Firstly install kind :

> Kind : Kubernetes in docker, tool that lets us run a full Kubernetes cluster inside Docker containers.
Gist : mini Kubernetes playground
> 

```solidity
Brew install kind
```

Then create a cluster

```solidity
kind create cluster --name signoz-shivam
```

<div class="home-hero">
  <img src="/images/image-21.png">
  <div class="home-quote">
  </div> 
 </div>


And once we do this, it creates a bunch of pods, these pods are ecs( for key value pair), deployment etc.
These are essential for the master mode to work properly and manage the worker nodes.<Br>
And we can also run a few pods of our own. 

So we do this by running: 


```python
kubectl apply -f podname.yaml
```



> What is Kubectl <br>
→Kubernetes CLI, which facilitates running commands against your Kubernetes cluster.
> 

And then we get list of all pods here, by running:

```solidity
Kubectl get pods -A
```

<div class="home-hero">
  <img src="/images/image-1.png">
  <div class="home-quote">
  </div> 
 </div>

Then,

```solidity
Brew install helm
```

> Helm is Kubernetes packet manager, for installing kubernetes stuff.
> 

Then add Signoz helm repo :

```solidity
helm repo add signoz https://charts.signoz.io
```

Now, as we are using Signoz cloud, for the generic case ( just simple K8s pods), we need to send data from K8s to Signoz, and here, we will use K8s-infra for that.

```
There are 3 main Collection agents ( yeah, which facilitate sending data from 
cluster to Signoz cloud)
1)K8s-Infra ( using Helm chart)
2)OpenTelemetry Operator
3)K8s serverless(EKS fargate)
```

Overview: <br>
So here, K8s-Infra deploys, otelAgent( pod level), and otelDeployment(cluster level), in one shot.

OtelAgent : deployed as Daemonset, i.e. runs on every node of cluster, to collect node and pod level data, like CPU, memory usage, state of each pod.<Br>
OtelDeployment : to gather cluster metrics.

Crazy visualtion of how it works :

<div class="home-hero">
  <img src="/images/image-2.png">
  <div class="home-quote">
  </div> 
 </div>

<div class="home-hero">
  <img src="/images/image-3.png">
  <div class="home-quote">
  </div> 
 </div>

After this, collector is installed and will automatically send telemetry data from your cluster and send them to SigNoz.

We will use k8s infra for sending the data of the kubernetes, there are 2 options - either this or opentelemetry collector and Installer.

K8s-infra installs telemetry operator and collector.

> Brief about K8s-Infra :
Collects data from cluster and sends to Signoz, acts as gateway to send any incoming OTLP telemetry data to SigNoz OtelCollector.
K8s-Infra are not different from OpenTelemetry operator part, it just abstracts the part of deploying 2 OpenTelemetry collectors.
> 

```
Observability → logs, metrics and traces.
Metrics. —  numeric measurements of system state over time (e.g., 
CPU, memory, request latency)
Logs -  detailed event records of what happened inside a system (e.g.,
 errors, print/debug messages)
Traces — end-to-end path of a request across services, showing 
latency and bottlenecks
 
```

The commands are this — you have to configure a yaml file here, so we are basically installing from the helm repo of signoz, and it has some default values, and while downloading we download that.

So we override those values, by upgrading it with some pre configured metrics which we want to send. The metrics here are details about the kubernetes clusters and pods, which bh default are not enabled.

Install the release by overriding some of the default values of OtelCollector ( here we want Kubernetes cluster data, which we have to enable)

So, we have this file override-values.yaml :
This is the main part, according to our use case, like here, we just have Kubernetes pods, if we have some applications of some language, we have to set the override-values.yaml file as required.

override-values.yaml

```yaml
global:
  cloud: others
  clusterName: kind-signoz-shivam
  deploymentEnvironment: dev

otelCollectorEndpoint: ingest.in.signoz.cloud:443
otelInsecure: false
signozApiKey: <SIGNOZ_INGESTION_KEY>

presets:
  otlpExporter:
    enabled: true

  # Node/pod/container metrics from each node (DaemonSet)
  kubeletMetrics:
    enabled: true
    collectionInterval: 30s
    insecureSkipVerify: true   

  # Cluster-wide metrics & events (Deployment) — gives you k8s_pod_phase, etc.
  clusterMetrics:
    enabled: true
    metrics:
      k8s.pod.phase:
        enabled: true
      k8s.pod.status_reason:   
        enabled: true
      k8s.namespace.phase:     
        enabled: true

  # Useful extras:
  kubernetesAttributes:
    enabled: true
  hostMetrics:
    enabled: true
  logsCollection:
    enabled: true
  k8sEvents:
    enabled: true

```

1. Then install k8s-infra chart with above config :

```solidity
helm install k8s-infra signoz/k8s-infra -n observability --create-namespace 
-f override-values.yaml
```

<div class="home-hero">
  <img src="/images/image-4.png">
  <div class="home-quote">
  </div> 
 </div>

And once we deploy the k8s, we are ready to send data in our signoz dashboard.

Signoz part :<bR>
Go to metrics and then create dashboard.<BR>
Now we will create different metrics <BR>

<strong>So we create metrics for :</strong>
```
A)State of each pod ( pending , running)
B)CPU utilization pod wise 
C)Memory usage
D)Network errors of each pod
E)Per pod network traffic
```

A) State of each pod 

<div class="home-hero">
  <img src="/images/image-5.png">
  <div class="home-quote">
  </div> 
 </div>

The pods running are represented as 2, and pods in pending state values return 1. 

Code is this :<BR>
So we get pod status for each pod, if return is 1, it means pending, and if return is 2, it means running.

<div class="home-hero">
  <img src="/images/image-6.png">
  <div class="home-quote">
  </div> 
 </div>

So here we have 11 running and 1 pending pod.<BR>
We can compare this by running our command in terminal :

<div class="home-hero">
  <img src="/images/image-7.png">
  <div class="home-quote">
  </div> 
 </div>

Which matches with running and pending pods!

B) CPU utilization pod wise 

<div class="home-hero">
  <img src="/images/image-8.png">
  <div class="home-quote">
  </div> 
 </div>

Code :

<div class="home-hero">
  <img src="/images/image-9.png">
  <div class="home-quote">
  </div> 
 </div>

C)Memory usage

<div class="home-hero">
  <img src="/images/image-10.png">
  <div class="home-quote">
  </div> 
 </div>

Code :

<div class="home-hero">
  <img src="/images/image-11.png">
  <div class="home-quote">
  </div> 
 </div>

D)Network errors of each pod

<div class="home-hero">
  <img src="/images/image-12.png">
  <div class="home-quote">
  </div> 
 </div>

<div class="home-hero">
  <img src="/images/image-13.png">
  <div class="home-quote">
  </div> 
 </div>

E)Per pod network traffic

<div class="home-hero">
  <img src="/images/image-14.png">
  <div class="home-quote">
  </div> 
 </div>

<div class="home-hero">
  <img src="/images/image-15.png">
  <div class="home-quote">
  </div> 
 </div>

# Creating an Alert

<div class="callout">
<strong> 💡 Now we will create an alert, we will fill the alert dashboard with the required values.</strong>
</div>

```
What we are creating an alert for -
If any of the pods are in pending state for >5 mins, then alert will be created.
```


Here is how we will fill out alert page : <BR>
→ Here we have selected k8s.pod.phase as metric, in the cluster - ‘kind-signoz-shivam’,
where we filter by k8s.pod.name<BR>
And for the Alert, we config that send a notification when value of k8s.pod.phase is equal to 1( which points to the pending state), during the last 5 mins.

<div class="home-hero">
  <img src="/images/image-16.png">
  <div class="home-quote">
  </div> 
 </div>

<div class="home-hero">
  <img src="/images/image-17.png">
  <div class="home-quote">
  </div> 
 </div>

<div class="home-hero">
  <img src="/images/image-18.png">
  <div class="home-quote">
  </div> 
 </div>

Now when we run the query we can see:

<div class="home-hero">
  <img src="/images/image-19.png">
  <div class="home-quote">
  </div> 
 </div>

When the threshold equals 1→ pending pod demo

For other pods →

<div class="home-hero">
  <img src="/images/image-20.png">
  <div class="home-quote">
  </div> 
 </div>

You can see the value is high up.

So we have setup our whole thing here.

Links :<br>
[https://signoz.io/blog/opentelemetry-kubernetes/](https://signoz.io/blog/opentelemetry-kubernetes/)<br>
[https://signoz.io/blog/kubectl-top/](https://signoz.io/blog/kubectl-top/)<br>
[https://signoz.io/blog/kubernetes-observability-with-opentelemetry/](https://signoz.io/blog/kubernetes-observability-with-opentelemetry/)<Br>
[https://signoz.io/blog/kubernetes-events-monitoring/](https://signoz.io/blog/kubernetes-events-monitoring/)<Br>
[https://signoz.io/blog/using-signoz-to-monitor-your-kubernetes-cluster/](https://signoz.io/blog/using-signoz-to-monitor-your-kubernetes-cluster/)<br>
[https://signoz.io/blog/opentelemetry-kubernetes-cluster-metrics-monitoring/](https://signoz.io/blog/opentelemetry-kubernetes-cluster-metrics-monitoring/)<Br>
[https://signoz.io/blog/kubernetes-logging/](https://signoz.io/blog/kubernetes-logging/)<Br>
[https://signoz.io/docs/collection-agents/get-started/](https://signoz.io/docs/collection-agents/get-started/)<Br>
[https://signoz.io/docs/collection-agents/k8s/k8s-infra/overview/](https://signoz.io/docs/collection-agents/k8s/k8s-infra/overview/)<br>
[https://signoz.io/docs/collection-agents/k8s/k8s-infra/configure-k8s-infra/](https://signoz.io/docs/collection-agents/k8s/k8s-infra/configure-k8s-infra/)<Br>
[https://signoz.io/docs/userguide/send-metrics-cloud/](https://signoz.io/docs/userguide/send-metrics-cloud/)<Br>