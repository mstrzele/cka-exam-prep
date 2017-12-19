# Certified Kubernetes Administrator (CKA) Exam Preparation

## 5% - Scheduling

- [x] [Use label selectors to schedule Pods.](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)
  * [`nodeSelector`](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)
    
    > It specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels (it can have additional labels as well).

    > ```bash
    > kubectl label nodes <node-name> <label-key>=<label-value>
    > ```

    > ```yaml
    > apiVersion: v1
    > kind: Pod
    > metadata:
    >   name: nginx
    >   labels:
    >     env: test
    > spec:
    >   containers:
    >   - name: nginx
    >     image: nginx
    >     imagePullPolicy: IfNotPresent
    >   nodeSelector:
    >     disktype: ssd
    > ```

    > ##### Interlude: built-in node labels
    > 
    > * `kubernetes.io/hostname`
    > * `failure-domain.beta.kubernetes.io/zone`
    > * `failure-domain.beta.kubernetes.io/region`
    > * `beta.kubernetes.io/instance-type`
    > * `beta.kubernetes.io/os`
    > * `beta.kubernetes.io/arch`

  * [Affinity and anit-affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)

    > The key enhancements are
    > 1. the language is more expressive (not just “AND of exact match”)
    > 2. you can indicate that the rule is “soft”/”preference” rather than a hard requirement, so if the scheduler can’t satisfy it, the pod will still be scheduled
    > 3. you can constrain against labels on other pods running on the node (or other topological domain), rather than against labels on the node itself, which allows rules about which pods can and cannot be co-located

    * [Node affinity (beta feature)](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#node-affinity-beta-feature)

      > Node affinity is conceptually similar to `nodeSelector` – it allows you to constrain which nodes your pod is eligible to schedule on, based on labels on the node.
      > There are currently two types of node affinity, called 
      > * `requiredDuringSchedulingIgnoredDuringExecution`
      > * `preferredDuringSchedulingIgnoredDuringExecution`
      
      > The “IgnoredDuringExecution” part of the names means that, similar to how `nodeSelector` works, if labels on a node change at runtime such that the affinity rules on a pod are no longer met, the pod will still continue to run on the node.

      > ```yaml
      > spec:
      >   affinity:
      >     nodeAffinity:
      >       requiredDuringSchedulingIgnoredDuringExecution:
      >         nodeSelectorTerms:
      >         - matchExpressions:
      >           - key: kubernetes.io/e2e-az-name
      >             operator: In
      >             values:
      >             - e2e-az1
      >             - e2e-az2
      >       preferredDuringSchedulingIgnoredDuringExecution:
      >       - weight: 1
      >         preference:
      >           matchExpressions:
      >           - key: another-node-label-key
      >             operator: In
      >             values:
      >             - another-node-label-value
      > ```

      > The new node affinity syntax supports the following operators:
      > * `In`,
      > * `NotIn`,
      > * `Exists`,
      > * `DoesNotExist`,
      > * `Gt`,
      > * `Lt`.

      > If you specify both `nodeSelector` and `nodeAffinity`, _both_ must be satisfied for the pod to be scheduled onto a candidate node.

      > If you specify multiple `nodeSelectorTerms` associated with `nodeAffinity` types, then the pod can be scheduled onto a node **if one of** the `nodeSelectorTerms` is satisfied.

      > If you specify multiple `matchExpressions` associated with `nodeSelectorTerms`, then the pod can be scheduled onto a node **only if all** `matchExpressions` can be satisfied.

    * [Inter-pod affinity and anti-affinity (beta feature)](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity-beta-feature)
      
      > Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled *based on labels on pods that are already running on the node* rather than based on labels on nodes.

      > * `requiredDuringSchedulingIgnoredDuringExecution`
      > * `preferredDuringSchedulingIgnoredDuringExecution`

      > You express it using a `topologyKey` which is the key for the node label that the system uses to denote such a topology domain

      > ```yaml
      > spec:
      >   affinity:
      >     podAffinity:
      >       requiredDuringSchedulingIgnoredDuringExecution:
      >       - labelSelector:
      >           matchExpressions:
      >           - key: security
      >             operator: In
      >             values:
      >             - S1
      >         topologyKey: failure-domain.beta.kubernetes.io/zone
      >     podAntiAffinity:
      >       preferredDuringSchedulingIgnoredDuringExecution:
      >       - weight: 100
      >         podAffinityTerm:
      >           labelSelector:
      >             matchExpressions:
      >             - key: security
      >               operator: In
      >               values:
      >               - S2
      >           topologyKey: kubernetes.io/hostname
      > ```

      > The legal operators for pod affinity and anti-affinity are
      > * `In`,
      > * `NotIn`,
      > * `Exists`,
      > * `DoesNotExist`
- [x] [Understand the role of DaemonSets.](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
  * [What is a DaemonSet?](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#what-is-a-daemonset)

    > A *DaemonSet* ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected.

    > Some typical uses of a DaemonSet are:
    > * running a cluster storage daemon, such as `glusterd`, `ceph`, on each node.
    > * running a logs collection daemon on every node, such as `fluentd` or `logstash`.
    > * running a node monitoring daemon on every node, such as Prometheus Node Exporter, `collectd`, Datadog agent, New Relic agent, or Ganglia `gmond`.

  * [Writing a DaemonSet Spec](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec)

    > ```yaml
    > apiVersion: apps/v1beta2 # for versions before 1.8.0 use extensions/v1beta1
    > kind: DaemonSet
    > ```

    > A Pod Template in a DaemonSet must have a `RestartPolicy` equal to `Always`, or be unspecified, which defaults to `Always`.

    > As of Kubernetes 1.8, you must specify a pod selector that matches the labels of the `.spec.template`.

  * [How Daemon Pods are Scheduled](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#how-daemon-pods-are-scheduled)

    > If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will create Pods on nodes which match that node selector. Likewise if you specify a `.spec.template.spec.affinity`, then DaemonSet controller will create Pods on nodes which match that node affinity.
    > * The `unschedulable` field of a node is not respected by the DaemonSet controller.
    > * The DaemonSet controller can make Pods even when the scheduler has not been started, which can help cluster bootstrap.

    > Daemon Pods do respect taints and tolerations, but they are created with `NoExecute` tolerations for the following taints with no `tolerationSeconds`:
    > * `node.kubernetes.io/not-ready`
    > * `node.alpha.kubernetes.io/unreachable`

  * [Updating a DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#updating-a-daemonset)

    > If node labels are changed, the DaemonSet will promptly add Pods to newly matching nodes and delete Pods from newly not-matching nodes.

    > In Kubernetes version 1.6 and later, you can perform a rolling update on a DaemonSet.

  * [Alternatives to DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#alternatives-to-daemonset)

    * Init Scripts
    * Bare Pods
    * Static Pods
    * Deployments
- [ ] Understand how resource limits can affect Pod scheduling.
- [ ] Understand how to run multiple schedulers and how to configure Pods to use them.
- [ ] Manually schedule a pod without a scheduler.
- [ ] Display scheduler events.
- [ ] Know how to configure the Kubernetes scheduler.

## 5% - Logging/Monitoring
 
- [ ] Understand how to monitor all cluster components.
- [ ] Understand how to monitor applications.
- [ ] Manage cluster component logs.
- [ ] Manage application logs.

## 8% - Application Lifecycle Management

- [ ] Understand Deployments and how to perform rolling updates and rollbacks.
- [ ] Know various ways to configure applications.
- [ ] Know how to scale applications.
- [ ] Understand the primitives necessary to create a self-healing application.

## 11% - Cluster Maintenance

- [ ] Understand Kubernetes cluster upgrade process.
- [ ] Facilitate operating system upgrades.
- [ ] Implement backup and restore methodologies.

## 12% - Security

- [ ] Know how to configure authentication and authorization.
- [ ] Understand Kubernetes security primitives.
- [ ] Know to configure network policies.
- [ ] Create and manage TLS certificates for cluster components.
- [ ] Work with images securely.
- [ ] Define security contexts.
- [ ] Secure persistent key value store.
- [ ] Work with role-based access control.

## 7% - Storage

- [ ] Understand persistent volumes and know how to create them.
- [ ] Understand access modes for volumes.
- [ ] Understand persistent volume claims primitive.
- [ ] Understand Kubernetes storage objects.
- [ ] Know how to configure applications with persistent storage.

## 10% - Troubleshooting

- [ ] Troubleshoot application failure.
- [ ] Troubleshoot control plane failure.
- [ ] Troubleshoot worker node failure.
- [ ] Troubleshoot networking.

## 19% - Core Concepts

- [ ] Understand the Kubernetes API primitives.
- [ ] Understand the Kubernetes cluster architecture.
- [ ] Understand Services and other network primitives.

## 11% - Networking

- [ ] Understand the networking configuration on the cluster nodes.
- [ ] Understand Pod networking concepts.
- [ ] Understand service networking.
- [ ] Deploy and configure network load balancer.
- [ ] Know how to use Ingress rules.
- [ ] Know how to configure and use the cluster DNS.
- [ ] Understand CNI.

## 12% - Installation, Configuration & Validation

- [ ] Design a Kubernetes cluster.
- [ ] Install Kubernetes masters and nodes, including the use of TLS bootstrapping.
- [ ] Configure secure cluster communications.
- [ ] Configure a Highly-Available Kubernetes cluster.
- [ ] Know where to get the Kubernetes release binaries.
- [ ] Provision underlying infrastructure to deploy a Kubernetes cluster.
- [ ] Choose a network solution.
- [ ] Choose your Kubernetes infrastructure configuration.
- [ ] Run end-to-end tests on your cluster.
- [ ] Analyse end-to-end tests results.
- [ ] Run Node end-to-end tests.
