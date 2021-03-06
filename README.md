<a name="iapetus"></a>
# Iapetus 🔱

[![Travis build status](http://img.shields.io/travis/gajus/iapetus/master.svg?style=flat-square)](https://travis-ci.org/gajus/iapetus)
[![Coveralls](https://img.shields.io/coveralls/gajus/iapetus.svg?style=flat-square)](https://coveralls.io/github/gajus/iapetus)
[![NPM version](http://img.shields.io/npm/v/iapetus.svg?style=flat-square)](https://www.npmjs.org/package/iapetus)
[![Canonical Code Style](https://img.shields.io/badge/code%20style-canonical-blue.svg?style=flat-square)](https://github.com/gajus/canonical)
[![Twitter Follow](https://img.shields.io/twitter/follow/kuizinas.svg?style=social&label=Follow)](https://twitter.com/kuizinas)

[Prometheus](https://prometheus.io/) metrics server.

* [Iapetus 🔱](#iapetus)
    * [Behaviour](#iapetus-behaviour)
        * [User-defined metrics](#iapetus-behaviour-user-defined-metrics)
        * [Default metrics](#iapetus-behaviour-default-metrics)
        * [`/metrics`](#iapetus-behaviour-metrics)
    * [Usage](#iapetus-usage)
        * [Kubernetes configuration](#iapetus-usage-kubernetes-configuration)
        * [Logging](#iapetus-usage-logging)
    * [FAQ](#iapetus-faq)
        * [Where does the name _Iapetus_ come from?](#iapetus-faq-where-does-the-name-_iapetus_-come-from)
        * [What is the difference from `prom-client`?](#iapetus-faq-what-is-the-difference-from-prom-client)


<a name="iapetus-behaviour"></a>
## Behaviour

Creates a HTTP service on port 9050 (default) with a [`/metrics`](#metrics) endpoint exposing [default metrics](#default-metrics) and [user-defined metrics](#user-defined-metrics).

<a name="iapetus-behaviour-user-defined-metrics"></a>
### User-defined metrics

There are 4 types of metrics to choose from: counter, gauge, summary and histogram. See the Prometheus documentation on [metric types](https://prometheus.io/docs/concepts/metric_types/) and [best practices](https://prometheus.io/docs/practices/instrumentation/#counter-vs.-gauge,-summary-vs.-histogram) for when to use which.

<a name="iapetus-behaviour-user-defined-metrics-counter"></a>
#### Counter

A counter is a cumulative metric that represents a single monotonically increasing counter whose value can only increase or be reset to zero on restart. For example, you can use a counter to represent the number of requests served, tasks completed, or errors.

Do not use a counter to expose a value that can decrease. For example, do not use a counter for the number of currently running processes; instead use a gauge.

```js
export type CounterMetricConfigurationType = {|
  +description: string,
  +labelNames?: $ReadOnlyArray<string>,
  +name: string
|};

/**
 * @property timestamp Time in milliseconds.
 */
type CounterMetricPayloadType = {|
  +labels?: LabelsType,
  +timestamp?: number,
  +value: number
|};

export type CounterMetricType = {|
  +increment: (payload?: CounterMetricPayloadType) => void
|};

```

<a name="iapetus-behaviour-user-defined-metrics-gauge"></a>
#### Gauge

A gauge is a metric that represents a single numerical value that can arbitrarily go up and down.

Gauges are typically used for measured values like temperatures or current memory usage, but also "counts" that can go up and down, like the number of running active HTTP requests.

```js
export type GaugeMetricConfigurationType = {|
  +description: string,
  +labelNames?: $ReadOnlyArray<string>,
  +name: string
|};

/**
 * @property timestamp Time in milliseconds.
 */
type GaugeMetricPayloadType = {|
  +labels?: LabelsType,
  +timestamp?: number,
  +value: number
|};

export type GaugeMetricType = {|
  +decrement: (payload?: GaugeMetricPayloadType) => void,
  +increment: (payload?: GaugeMetricPayloadType) => void,
  +set: (payload: GaugeMetricPayloadType) => void
|};

```

<a name="iapetus-behaviour-user-defined-metrics-summary"></a>
#### Summary

N/A

<a name="iapetus-behaviour-user-defined-metrics-histogram"></a>
#### Histogram

N/A

<a name="iapetus-behaviour-default-metrics"></a>
### Default metrics

The following metrics are collected and reported by default:

|Name|Labels|Description|
|---|---|---|
|`process_cpu_user_seconds_total`||Total user CPU time spent in seconds.|
|`process_cpu_system_seconds_total`||Total system CPU time spent in seconds.|
|`process_cpu_seconds_total`||Total user and system CPU time spent in seconds.|
|`process_start_time_seconds`||Start time of the process since unix epoch in seconds.|
|`process_resident_memory_bytes`||Resident memory size in bytes.|
|`nodejs_eventloop_lag_seconds`||Lag of event loop in seconds.|
|`nodejs_active_handles_total`||Number of active handles.|
|`nodejs_active_requests_total`||Number of active requests.|
|`nodejs_heap_size_total_bytes`||Process heap size from node.js in bytes.|
|`nodejs_heap_size_used_bytes`||Process heap size used from node.js in bytes.|
|`nodejs_external_memory_bytes`||Nodejs external memory size in bytes.|
|`nodejs_heap_space_size_total_bytes`|`space`|Process heap space size total from node.js in bytes.|
|`nodejs_heap_space_size_used_bytes`|`space`|Process heap space size used from node.js in bytes.|
|`nodejs_heap_space_size_available_bytes`|`space`|Process heap space size available from node.js in bytes.|
|`nodejs_version_info`|`version`, `major`, `minor`, `patch`|Node.js version info.|
|`nodejs_gc_runs_total`|`gctype`|Count of total garbage collections.|
|`nodejs_gc_pause_seconds_total`|`gctype`|Time spent in GC Pause in seconds.|
|`nodejs_gc_reclaimed_bytes_total`|`gctype`|Total number of bytes reclaimed by GC.|

<!--
  const metrics = register.getMetricsAsArray();

  for (const metric of metrics) {
    console.log('|`' + metric.name + '`|' + (metric.labelNames.length ? '`' + metric.labelNames.join('`, `') + '`' : '') + '|' + metric.help + '|');
  }
-->

<a name="iapetus-behaviour-metrics"></a>
### <code>/metrics</code>

`/metrics` endpoint produces metrics for Prometheus consumption.

<a name="iapetus-usage"></a>
## Usage

Use `createIapetus` to create an instance of Iapetus.

```js
import {
  createIapetus
} from 'iapetus';

const configuration: IapetusConfigurationType = {};

const iapetus: IapetusType = createIapetus(configuration);

```

The following types describe the configuration shape and the resulting Iapetus instance interface.

```js
/**
 * @property port The port on which the Iapetus service listens. This port must be different than your main service port, if any. The default port is 9050.
 */
type IapetusConfigurationType = {|
  +port?: number
|};

/**
 * @property stop Stops the Iapetus server.
 */
type IapetusType = {|
  +createCounterMetric: (configuration: CounterMetricConfigurationType) => CounterMetricType,
  +createGaugeMetric: (configuration: GuageMetricConfigurationType) => GaugeMetricType,
  +getMetrics: () => $ReadOnlyArray<MetricDescriptorType>,
  +stop: () => Promise<void>
|};

```

<a name="iapetus-usage-kubernetes-configuration"></a>
### Kubernetes configuration

> This section assumes that you have installed Prometheus using Helm [Prometheus chart](https://github.com/helm/charts/tree/master/stable/prometheus).

Add the following annotations to your [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/):

```yaml
annotations:
  prometheus.io.scrape: 'true'
  prometheus.io.port: '9050'
  prometheus.io.path: '/metrics'

```

<a name="iapetus-usage-logging"></a>
### Logging

`iapetus` is using [Roarr](https://github.com/gajus/roarr) to implement logging.

Set `ROARR_LOG=true` environment variable to enable logging.

<a name="iapetus-faq"></a>
## FAQ

<a name="iapetus-faq-where-does-the-name-_iapetus_-come-from"></a>
### Where does the name <em>Iapetus</em> come from?

In Greek mythology, [Iapetus](https://en.wikipedia.org/wiki/Iapetus) (/aɪˈæpɪtəs/), was a Titan, the son of Uranus and Gaia, and father of Atlas, Prometheus, Epimetheus and Menoetius.

<a name="iapetus-faq-what-is-the-difference-from-prom-client"></a>
### What is the difference from <code>prom-client</code>?

Iapetus is a high-level abstraction of [`prom-client`](https://github.com/siimon/prom-client) with integrated HTTP server and pre-configured [default metrics](#default-metrics).
