---
title: A journey through quality of service in Kubernetes
author: nieldewet
---
The learning curve for mastering Kubernetes for your container orchestration can be quite steep if you don't have someone to guide you through it. Our approach was to first do the simplest thing that worked and to evolve it as the project progressed. This worked quite well initially but it came with several pitfalls. One of them is how Kubernetes deals with the different quality of service levels for pods and how that effects the scheduling of pods on your agents. <!--more-->

#### Context
We use a cloud-hosted Kubernetes cluster with a handful of agent VMs and only a couple of replicas for each service deployed.

#### The naive approach
Our first workloads were defined with **Best Effort** pods, but we didn't even know what that meant at the time. Initially, it was all fine until the workload and the number of pods began to grow, but when we deployed some resource-hungry applications to the cluster, our agent VMs would sometimes unexpectedly run out of CPU or memory resources and become completely unresponsive. The outages that this caused were disruptive because they often took down build tools that our team heavily relies on, like our continuous integration server or our Docker registry.

Your pods are Best Effort when you define your deployments without specifying any resource requests or limits. This means that the Kubernetes scheduler has no idea how to schedule them to fairly distribute the workload between different agents in the cluster and some really resource-intensive pods might end up on the same machine. 

As soon as these applications start to experience load (and they consume a large amount of resources, especially uncompressible resources like memory), the agent runs out of resources, your cluster becomes unstable and all your pods that were scheduled on that agent become unavailable.

Relying on Best Effort pods may be fine if all your containers are well behaved and use a comparable amount of resources. One of our neighbouring teams follows this approach very successfully and achieves quite high utilisation of resources. I wouldn't recommend this approach unless you understand your workload really well and, even then, I urge caution.

#### The conservative approach
After several nasty incidents where agents in our cluster fell over, we learned how to set resource limits. We studied the memory and CPU usage of our applications and set requests that matched typical usage, with limits that allowed for some spikes. Kubernetes assigns pods like these **Burstable** quality of service.

Kubernetes will schedule pods by attempting to optimise resources based on the request values and may oversubscribe resource limits. If your pods are tightly packed on a particular agent, you might still experience instability in your cluster if a couple of those pods start pushing up against the set limits, either by exhausting the agent's resources or because Kubernetes will kill pods that exceed their memory limits and throttle those that exceed CPU limits. After we ran into this scenario a couple of times, we tried to avoid it by setting request levels that were only 10% lower than the limits.

Finally, some stability! We felt we had finally successfully wrangled our still fairly small number of deployments, but our relief was short-lived. Soon there were not enough resources available on our agents to schedule more pods and we had to start scaling up the size and number of VMs in our cluster. This happened despite our VMs sitting idle most of the time! The utilisation of the VMs was shockingly low. That's because Kubernetes will not schedule a pod on an agent if adding it causes the sum of resource requests for pods on that agent to exceed its total schedulable resources.

#### Out of resource handling and guaranteed quality of service
Our current approach is to prescribe how our agents should deal with running out of resources and setting resource requests that align better with typical usage. The kubelet service can be configured with pods eviction rules based on memory and file system usage as a type of self-defence, with soft and hard limits, as well as grace periods and system-reserved memory. You can read more about the settings in the official [documentation](https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/). Here's an example of the current options that we are setting on the kubelet service:

    --eviction-soft=memory.available<10%,nodefs.available<2Gi \
	--eviction-soft-grace-period=memory.available=1m30s,nodefs.available=1m30s \
	--eviction-max-pod-grace-period=120 \
	--eviction-hard=memory.available<500Mi,nodefs.available<1Gi \
	--eviction-pressure-transition-period=30s \
	--system-reserved=memory=0.5Gi

Based on these settings, kubelet will evict pods when the agent starts running out of resources, allowing Kubernetes to reschedule them elsewhere, starting with **Best Effort** pods, followed by **Burstable** pods. **Guaranteed** pods are never evicted. Pods are Guaranteed if the resource request is equal to the resource limit.

To ensure uptime, we make applications that can not be replicated or take long to start up Guaranteed and increase the number of replicas for Burstable pods. We don't use Best Effort pods at all. We can now achieve higher utilisation of the agents because we don't have to set resource requests very high to compensate for spikes, but we do set some fairly high limits to guard against misbehaving pods. We are currently proving this approach and will continue to refine it.

> Written with [StackEdit](https://stackedit.io/).
