---
title: Integrate Keptn with your Applications
description: How to integrate Keptn into your Kubernetes cluster
weight: 200
---

Keptn works
on top of the default scheduler for the cluster,
so it can trace all activities of all deployment workloads on the cluster,
no matter what tool is used for the deployment.
This same mechanism allows Keptn to inject pre- and post-deployment checks
into all deployment workloads.
Keptn monitors resources
that have been applied into the Kubernetes cluster
and reacts if it finds a workload with special annotations/labels.
Keptn uses metadata
that is added to the Kubernetes workloads
to identify the workloads of interest.

To integrate Keptn with your applications:

* You must first
[install and enable](../installation/_index.md#basic-installation)
Keptn.
* Annotate or label your workloads
with either Keptn or Kubernetes keys.
  * [Basic annotations](#basic-annotations)
    or labels
    are required for all Keptn features except Keptn metrics.
  * [Pre- and post-deployment checks](#basic-annotations)
    are required only for the Release lifecycle management feature.

Keptn uses these annotations to the Kubernetes workloads to create the
[KeptnWorkload](../reference/api-reference/lifecycle/v1alpha3/#keptnworkload)
and
[KeptnApp](../reference/crd-reference/app.md)
resources that it uses to provide observability
and release lifecycle management.

> Note: Annotations are not required if you are only using the
  `metrics-operator` component of Keptn
  to observe Keptn metrics.

## Basic annotations

Keptn automatically discovers `KeptnApp` resources,
based on the annotations or labels.
This enables the Keptn observability features
(based on OpenTelemetry) for existing applications,
without additional Keptn configuration.

Keptn monitors your
[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/),
[StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/),
and
[ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/),
and
[DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
resources in the namespaces where Keptn is enabled.
If Keptn finds any of these resources and the resource has either
the `keptn.sh` or the `kubernetes` annotations/labels,
it creates appropriate
[KeptnWorkload](../reference/api-reference/lifecycle/v1alpha3/#keptnworkload)
and
[KeptnApp](../reference/crd-reference/app.md)
resources for the version it detects.

The basic keptn.sh keys that can be used for annotations or labels are:

```yaml
keptn.sh/workload: myAwesomeWorkload
keptn.sh/version: myAwesomeWorkloadVersion
keptn.sh/app: myAwesomeAppName
keptn.sh/container: myAwesomeContainer
```

Alternatively, you can use Kubernetes keys for annotations or labels.
These are part of the Kubernetes
[Recommended Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/):

```yaml
app.kubernetes.io/name: myAwesomeWorkload
app.kubernetes.io/version: myAwesomeWorkloadVersion
app.kubernetes.io/part-of: myAwesomeAppName
```

These keys are defined as:

* `keptn.sh/workload` or `app.kubernetes.io/name`: Determines the name
  of the generated
  [KeptnWorkload](../reference/api-reference/lifecycle/v1alpha3/#keptnworkload)
  resource.
* `keptn.sh/version` or `app.kubernetes.io/version`:
  Determines the version of the `KeptnWorkload`
  that represents the Workload.
  If the Workload has no `version` annotation/labels
  and the pod has only one container,
  Keptn takes the image tag as version
  (unless it is "latest").
* `keptn.sh/app` or `app.kubernetes.io/part-of`: Determines the name
   of the generated `KeptnApp` representing your Application.
   All workloads that share the same value for this label
   are consolidated into the same `KeptnApp` resource.
* `keptn.sh/container`: Determines the name of the container in the workload,
   from which Keptn extracts the version.
   This applies to single- and multi-container
   workloads.
   If the given container name does not match any container in the workload
   no version can be determined.
   Note that there is no equivalent `app.kubernetes.io/` annotation/label for this label.

Keptn automatically generates appropriate
[KeptnApp](../reference/crd-reference/app.md)
resources that are used for observability,
based on whether the `keptn.sh/app` or `app.kubernetes.io/part-of`
annotation/label is populated:

* If either of these annotations/labels are populated,
  Keptn automatically generates a `KeptnApp` resource
  that includes all workloads that have the same annotation/label,
  thus creating a `KeptnApp` resource for each defined grouping

* If only the `workload` and `version` annotations/labels are available
  (in other words, neither the `keptn.sh/app`
  or `app.kubernetes.io/part-of` annotation/label is populated),
  Keptn creates a `KeptnApp` resource for each `KeptnWorkload`
  and your observability output traces the individual `Keptnworkload` resources
  but not the combined workloads that constitute your deployed application.

See
[Keptn Applications and Keptn Workloads](../components/lifecycle-operator/keptn-apps.md)
for architectural information about how `KeptnApp` and `KeptnWorkloads`
are implemented.

## Annotations vs. labels

The same keys can be used as
[annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)
or
[labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).
Both annotations and labels are can be attached to Kubernetes objects.
Some key differences between the two:

* Annotations
  * Are not used to identify and select objects
  * Can contain up to 262144 chars
  * Metadata in an annotation can be small or large,
    structured or unstructured,
    and can include characters not permitted by labels

* Labels
  * Can be used to select objects
    and to find collections of objects that satisfy certain conditions
  * Can contain up to 63 chars
  * Are appropriate for identifying attributes of objects
    that are meaningful and relevant to users
    but do not directly imply semantics to the core system

Annotations take precedence over labels,
and the `keptn.sh` keys take precedence over `app.kubernetes.io` keys.
In other words:

* The operator first checks if the `keptn.sh` key is present
  in the annotations, and then in the labels.
* If neither is the case, it looks for the `app.kubernetes.io` equivalent,
  again first in the annotations, then in the labels.

In general, annotations are more appropriate than labels
for integrating Keptn with your applications
because they store references, names, and version information
so the 63 char limitation is quite restrictive.
However, labels can be used if you specifically need them
and can accommodate the size restriction.

## Pre- and post-deployment checks

To implement the Keptn Release Lifecycle feature
that handles pre- and post-deployment evaluations and tasks,
do the following:

* Define the
  [KeptnMetric](../reference/crd-reference/metric.md)
  and
  [KeptnEvaluationDefinition](../reference/crd-reference/evaluationdefinition.md)
  resources for each evaluation you want.
  A `KeptnEvaluationDefinition` compares the value
  of a `KeptnMetric` to the threshold that is specified.
* You will also need to define the necessary
  [KeptnMetricsProvider](../reference/crd-reference/metricsprovider.md)
  and
  resource for each instance of each data source
  used for the `KeptnEvaluationDefinition` resources you define.
* Define a
  [KeptnTaskDefinition](../reference/crd-reference/taskdefinition.md)
  resource for each task you want to execute.
  `KeptnTaskDefinition`  resources contain re-usable "functions"
  that can execute before and after the deployment.
  For example, before the deployment starts,
  you might run a check for open problems in your infrastructure
  and invoke a pipeline to run performance tests.
  The deployment is kept in a pending state
  until the infrastructure is capable of accepting deployments again.
  See
  [Working with Keptn tasks](./tasks.md)
  for more information.
* Annotate your [Workloads](https://kubernetes.io/docs/concepts/workloads/)
  [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/),
  [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/),
  and
  [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
  to include each evaluation and task you want run
  for specific workloads.
* Manually edit all
  [KeptnApp](../reference/crd-reference/app.md) resources
  to specify evaluations and tasks to be run for the `KeptnApp` itself.

### Annotations to KeptnApp

The annotations to workloads do not define the tasks and evaluations
to be run for `KeptnApp` resources themselves.
To define pre- and post-deployment evaluations and tasks for a `KeptnApp`,
you must manually edit the YAML file to add them.

Specify one of the following annotations/labels
for each evaluation or task you want to execute:

```yaml
keptn.sh/pre-deployment-evaluations: <`EvaluationDefinition`-name>
keptn.sh/pre-deployment-tasks: <`TaskDefinition`-name>
keptn.sh/post-deployment-evaluations: <`EvaluationDefinition`-name>
keptn.sh/post-deployment-tasks: <`TaskDefinition`-name>
```

The value of these annotations corresponds to the name of
Keptn [resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
called
[KeptnTaskDefinition](../reference/crd-reference/taskdefinition.md)
resources
These resources contain re-usable "functions"
that can execute before and after the deployment.
For example, before the deployment starts,
you might run a check for open problems in your infrastructure
and invoke a pipeline to run performance tests.
The deployment is kept in a pending state
until the infrastructure is capable of accepting deployments again.

If everything is fine, the deployment continues and afterward,
a Slack notification is sent with the result of the deployment

## Example of pre- and post-deployment actions

A comprehensive example of pre- and post-deployment
evaluations and tasks can be found in our
[examples folder](https://github.com/keptn/lifecycle-toolkit/tree/main/examples/sample-app),
where we use [Podtato-Head](https://github.com/podtato-head/podtato-head)
to run some simple pre-deployment checks.

To run the example, use the following commands:

```shell
cd ./examples/podtatohead-deployment/
kubectl apply -f .
```

Afterward, you can monitor the status of the deployment using

```shell
kubectl get keptnworkloadversion -n podtato-kubectl -w
```

The deployment for a Workload stays in a `Pending`
state until the respective pre-deployment check is successfully completed.
Afterwards, the deployment starts and when the workload is deployed,
the post-deployment checks start.