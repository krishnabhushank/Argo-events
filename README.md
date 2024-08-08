# Argo-events

In Argo Events, you can chain event flows by setting up triggers that initiate the next event source. This can be done using custom triggers that create or update event sources as part of the trigger's action. Here’s how you can achieve this:

### Step-by-Step Guide

#### 1. Define Initial Event Source
Create an initial event source that will start the event chain.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: EventSource
metadata:
  name: initial-event-source
spec:
  webhooks:
    example:
      endpoint: /example
      method: POST
      port: "12000"
```

#### 2. Create the First Sensor
Create a sensor that listens to the initial event source and triggers the creation of the next event source.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Sensor
metadata:
  name: first-sensor
spec:
  dependencies:
    - name: initial-event
      eventSourceName: initial-event-source
      eventName: example
  triggers:
    - template:
        name: create-next-event-source
        k8s:
          group: argoproj.io
          version: v1
          resource: configmaps
          operation: create
          source:
            resource:
              apiVersion: v1
              kind: ConfigMap
              metadata:
                name: next-event-source
              data:
                config.yaml: |
                  apiVersion: argoproj.io/v1alpha1
                  kind: EventSource
                  metadata:
                    name: next-event-source
                  spec:
                    webhooks:
                      example:
                        endpoint: /next-example
                        method: POST
                        port: "12001"
```

#### 3. Define the Next Event Source
The next event source can be created dynamically by the sensor. Here is the YAML definition that the sensor will trigger.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: EventSource
metadata:
  name: next-event-source
spec:
  webhooks:
    example:
      endpoint: /next-example
      method: POST
      port: "12001"
```

#### 4. Create the Second Sensor
Create a sensor that listens to the next event source and triggers the subsequent actions.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Sensor
metadata:
  name: next-sensor
spec:
  dependencies:
    - name: next-event
      eventSourceName: next-event-source
      eventName: example
  triggers:
    - template:
        name: trigger-workflow
        k8s:
          group: argoproj.io
          version: v1alpha1
          resource: workflows
          operation: create
          source:
            resource:
              apiVersion: argoproj.io/v1alpha1
              kind: Workflow
              metadata:
                generateName: example-workflow-
              spec:
                entrypoint: main
                templates:
                  - name: main
                    container:
                      image: my-image
                      command: ["sh", "-c"]
                      args: ["echo 'Next event triggered'"]
```

### Explanation

1. **Initial Event Source:** The initial event source listens for the first event (e.g., a webhook event).
2. **First Sensor:** The first sensor listens to the initial event source and, upon receiving an event, triggers the creation of the next event source. This is done using a Kubernetes resource trigger that creates a ConfigMap containing the next event source's definition.
3. **Next Event Source:** The next event source is created dynamically by the first sensor. This event source will listen for the next event in the chain.
4. **Next Sensor:** The next sensor listens to the next event source and triggers subsequent actions (e.g., running a workflow).

### Using the Argo Events UI

To visualize this chain in the Argo Events UI, follow these steps:

1. **Deploy the Argo Events UI:** Follow the [installation instructions](https://github.com/argoproj-labs/argo-events-ui) to set up the Argo Events UI.
2. **Access the UI:** Use port-forwarding or a service to access the UI in your browser.
3. **Visualize Event Sources and Sensors:** In the UI, you can see the event sources and sensors. The UI will show how the sensors are linked to the event sources, providing a graphical representation of the event chain.

### Monitoring and Observability

- **Logs:** Use `kubectl logs` to view logs for event sources and sensors.
- **Metrics:** Use Prometheus and Grafana to collect and visualize metrics.
- **Alerts:** Set up alerts using Prometheus Alertmanager for proactive monitoring.

By configuring triggers to create the next event source dynamically, you can effectively chain multiple event flows and visualize them using the Argo Events UI, enhancing the observability and manageability of your event-driven workflows.
