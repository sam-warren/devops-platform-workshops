# Persistent Configurations

<kbd>[![Video Walkthrough Thumbnail](././images/09_persistent_configurations_thumb.png)](https://youtu.be/g6TyE3rIHeo)</kbd>

[Video walkthrough](https://youtu.be/g6TyE3rIHeo)

In cases where configurations need to change frequently or common configurations should be shared across deployments or pods, it is not ideal to build said configurations into the container image or maintain multiple copies of the configuration. OpenShift supports `configMaps` which can be a standalone object that is easily mounted into pods. In cases where the configuration file or data is sensitive in nature, OpenShift supports `secrets` to handle this sensitive data.

### ConfigMaps

#### Creating a Config Map and Adding it to a Deployment

Create a configMap with arbitrary data and mount it inside of your `rocketchat-samwarren` pod:

<kbd>![](./images/07_persistent_config_01.png)</kbd>
<kbd>![](./images/07_persistent_config_02.png)</kbd>

- In the Web Console, go to `+Add` and select `YAML`
- Paste in the following ConfigMap Code, replace samwarren with your username and save

```yaml
apiVersion: v1
data:
  myfile.txt: Hello world
kind: ConfigMap
metadata:
  name: rocketchat-samwarren-configmap
  labels:
    app: rocketchat-samwarren
```

- Attach the `configMap` to your `rocketchat-samwarren` deployment by navigating to the `YAML` tab in your Rocket Chat Deployment and pasting in the following code

  1. You will first need to create a `volume`. This is located under `.spec.template.spec.volumes`

  ```yaml
  - name: rocketchat-samwarren-volume
       configMap:
         name: rocketchat-samwarren-configmap
  ```

  2. Then create a `volumeMount` under `.spec.template.spec.containers.volumeMounts`

  ```yaml
  - name: rocketchat-samwarren-volume
              mountPath: /opt/configs
  ```

  > pro tip: if you are not sure what fields are available you can always use oc explain! `oc explain deployment.spec.template.spec.containers.volumeMounts`

<kbd>![](./images/07_persistent_config_04.png)</kbd>
<kbd>![](./images/07_persistent_config_03.png)</kbd>

> it should like similar to this
> <kbd>![](./images/07_persistent_config_05.png)</kbd>

- Save the changes which will trigger a new deployment of your `rocketchat-samwarren` pod

- Using the pod terminal in the Web Console or `oc rsh`, explore the path of the mounted configMap

<kbd>![](./images/07_persistent_config_06.png)</kbd>

#### Changing Config Map Content

The content in your `configMap` can be changed and in most cases it's NOT dynamically updated in the pod. With that said, a new deployment will be required for the changes to be picked up.

- Edit your `configMap` to add a new key-value pair that represents another file

Navigate to `Administrator View -> Workloads -> Configmaps` from the console and select your ConfigMap from the list

<kbd>![](./images/07_persistent_config_06.png)</kbd>
<kbd>![](./images/07_persistent_config_07.png)</kbd>

- Using the pod terminal in the Web Console or `oc rsh`, explore the path of the mounted configMap

<kbd>![](./images/07_persistent_config_08.png)</kbd>

### Secrets

Secrets can be added in a similar way as config maps but are geared towards the management of sensitive information. In OpenShift, these are base64 encoded, and encrypted on disk when stored in the cluster. In Pods, they never live on disk (unlike configmaps) and are only in memory.
Secrets, from the Web Console, are focused on supporting:

- Username/Passwords
- SSH Keys
- SSL Certificates
- Git config files

"Opaque" secrets are supported and can contain any type of data, however, these must be configured on the command line with the `oc` cli.

### Creating Secrets

- In the Web Console, go to `+Add` and select `YAML`
- Paste in the following Secret Code and save

```yaml
apiVersion: v1
stringData:
  SECRET_API_KEY: "I'm ULTRA SECRET"
kind: Secret
metadata:
  creationTimestamp: null
  name: rocketchat-samwarren-secret
  labels:
    app: rocketchat-samwarren
```

<kbd>![](./images/07_persistent_config_09.png)</kbd>

- Explore the other mongo secrets to see different variations of secret data by navigating to `Administrator View -> Workloads -> Secrets`

<kbd>![](./images/07_persistent_config_10.png)</kbd>
<kbd>![](./images/07_persistent_config_11.png)</kbd>

- Navigate back to your `rocketchat-samwarren-secret` and attach the `secret` to your `rocketchat-samwarren` Deployment by navigating to the `Add Secret to Workload`

- Attach the secret as an environment variable

<kbd>![](./images/07_persistent_config_12.png)</kbd>
<kbd>![](./images/07_persistent_config_13.png)</kbd>

- This change will trigger a new deployment of your `rocketchat-samwarren` pod
- Once the pods have been redeployed the environment variable should be available for them to use. This is a very common pattern that applications can use to hold sensitive information like api keys in memory. You can verify that the environment variable exists by accessing the pod terminal or using `oc rsh` and outputting its value with a simple `echo $SECRET_API_KEY`

- From the cli, review the secret with `oc -n d8f105-dev describe secret rocketchat-samwarren-secret`

```
oc d8f105-dev describe secret rocketchat-samwarren-secret
Name:         rocketchat-samwarren-secret
Namespace:    [namespace]
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
SECRET_API_KEY:  16 bytes
```

- Export the secret to view the contents with `oc -n d8f105-dev get secret rocketchat-samwarren-secret -o yaml`

```
oc get -n d8f105-dev secret rocketchat-samwarren-secret -o yaml
apiVersion: v1
data:
  SECRET_API_KEY: SSdtIFVMVFJBIFNFQ1JFVA==
kind: Secret
metadata:
  creationTimestamp: null
  name: rocketchat-samwarren-secret
  selfLink: /api/v1/namespaces/[namespace]/secrets/rocketchat-samwarren-secret
type: Opaque
```

> on Mac's and Linux machines that have the base64 binary, you can decode the value as a reference

```
echo "SSdtIFVMVFJBIFNFQ1JFVA==" | base64 -d
```

- To edit an existing secret, from the webconsole you can navigate to the secret and select the `YAML` tab or `oc edit secret rocketchat-samwarren-secret` from the cli. Secrets are base64 encoded. If adding new values or editing existing values of a Secret it is often easier to
  edit in plain text and have it b64 encoded for you. To do this you will need to modify the `data` field and change it to
  `stringData` prior to adding or editing values.

- Add another secret value but this time as a sensitive configuration file such as an ssh key

From the web console
<kbd>![](./images/07_persistent_config_14.png)</kbd>
With `oc edit`
<kbd>![](./images/07_persistent_config_15.png)</kbd>

> take note that you were adding a new value `id_rsa` under a field called `stringData`. Openshift will automatically, encode that as base64 and place it in the `data` field upon saving. You can confirm this with `oc -n d8f105-dev get secret rocketchat-samwarren-secret -o yaml` after saving.

- Redeploy the application after secret changes

Next page - [Event Streams](./10_event_streams.md)
