# Persistent Storage

<kbd>[![Video Walkthrough Thumbnail](././images/08_persistent_storage_thumb.png)](https://youtu.be/yG_dzkUUYfg)</kbd>

[Video walkthrough](https://youtu.be/yG_dzkUUYfg)

Up to this point you have leveraged a single mongodb pod with ephemeral storage. In order to maintain the
application data, persistent storage is required.

Some background information about volumes and volumemounts can be found [here](https://kubernetes.io/docs/concepts/storage/volumes/).

- Let's first take a look at our application prior to this lab
  <kbd>![](./images/06_persistent_storage_01.png)</kbd>

### Deleting Pods with Ephemeral Storage

**Objective**: Observe that by using ephemeral storage causes RocketChat to lose any previous data or configuration after a redeployment.

To understand what will happen when a pod with ephemeral storage is removed,

- Scale down both the rocketchat and mongo applications to 0 pods
  ```oc:cli
  oc -n d8f105-dev scale deployment/rocketchat-samwarren dc/mongodb-samwarren --replicas=0
  ```
- Scale back up each application pod to 1 replica
  ```oc:cli
  oc -n d8f105-dev scale deployment/rocketchat-samwarren dc/mongodb-samwarren --replicas=1
  ```
  <kbd>![](./images/06_persistent_storage_02.png)</kbd>

### Adding Storage to Existing Deployment Configurations

**Objective**: Add persistent storage to MongoDB so that it won't lose data created by RocketChat.

Now that we notice all messages and configuration is gone whenever pods cycle, let's add persistent storage to the mongodb pod.

- Scale down both the rocketchat and mongo applications to 0 pods
  ```oc:cli
  oc -n d8f105-dev scale deployment/rocketchat-samwarren dc/mongodb-samwarren --replicas=0
  ```
- Remove the emptyDir Storage by navigating to the mongodb deploymentconfig
  <kbd>![](./images/06_persistent_storage_03.png)</kbd>

- Add a new volume by navigating to `Administrator -> Storage -> Persitant Volume Claims -> Create Persistant Volume Claims` and name it `mongodb-samwarren-file`

<kbd>![](./images/06_persistent_storage_04a.png)</kbd>

- Select the `netapp-file-standard` storage class. Set the type to RWO, the size to 1GB, select `Filesystem` mode, and name it `mongodb-samwarren-file`

- Navigate back to your Mongo DeploymentConfig and select `Add Storage` from the `Actions` menu

- The mount path is `/var/lib/mongodb/data`
  <kbd>![](./images/06_persistent_storage_04b.png)</kbd>

- Label your PVC

  ```
  oc -n d8f105-dev patch pvc mongodb-samwarren-file -p '{"metadata":{"labels":{"app":"rocketchat-samwarren"}}}'
  ```

- Scale up `mongodb-samwarren` instance to 1 pod
  ```oc:cli
  oc -n d8f105-dev scale dc/mongodb-samwarren --replicas=1
  ```
- When mongo is running, scale `rocketchat-samwarren` to 1 pod
  ```oc:cli
  oc -n d8f105-dev scale deployment/rocketchat-samwarren --replicas=1
  ```
- Access the RocketChat URL and complete the Setup Wizard again
- Scale down and scale back up both the database and the rocketchat app
  ```oc:cli
  oc -n d8f105-dev scale deployment/rocketchat-samwarren dc/mongodb-samwarren --replicas=0
  # Scale up MongoDB to 1 replica; and
  oc -n d8f105-dev scale dc/mongodb-samwarren --replicas=1
  # Scale up RocketChat to 1 replica
  oc -n d8f105-dev scale deployment/rocketchat-samwarren --replicas=1
  ```
- Verify that data was persisted by accessing RocketChat URL and observing that it doesn't show the Setup Wizard.

#### RWO Storage

RWO Storage is analagous to attaching a physical disk to a pod. For this reason, RWO storage **cannot be mounted to more than 1 pod at the same time**.

**Objective**: Cause deployment error by using the wrong deployment strategy for the storage class.

RWO storage (which was selected above) can only be attached to a single pod at a time, which causes issues in certain deployment strategies.

- Ensure your `mongodb-samwarren` deployment strategy is set to `Rolling' and then trigger a redeployment.

<kbd>![](./images/06_persistent_storage_07.png)</kbd>

- Notice and investigate the issue

> rolling deployments will start up a new version of your application pod before killing the previous one. There is a brief moment where two pods for the mongo application exist at the same time. Because the storage type is **RWO** it is unable to mount to two pods at the same time. This will cause the rolling deployment to hang and eventually time out.

<kbd>![](./images/06_persistent_storage_08.png)</kbd>

- Switch to recreate

### RWX Storage

**Objective**: Cause MongoDB to corrupt its data file by using the wrong storage class for MongoDB.

RWX storage allows multiple pods to access the same PV at the same time.

- Scale down `mongodb-samwarren` to 0 pods
  ```oc:cli
  oc -n d8f105-dev scale dc/mongodb-samwarren --replicas=0
  ```
- Create a new PVC as `netapp-file-standard', set the type to RWX and name it `mongodb-samwarren-file-rwx`
  <kbd>![](./images/06_persistent_storage_09.png)</kbd>

- Remove the previous storage volume and add your new `mongodb-samwarren-file-rwx` storage, mounting at `/var/lib/mongodb/data`

  <kbd>![](./images/06_persistent_storage_10.png)</kbd>

  ```oc:cli
  oc -n d8f105-dev rollout pause dc/mongodb-samwarren
  # Remove all volumes
  oc -n d8f105-dev get dc/mongodb-samwarren -o jsonpath='{.spec.template.spec.volumes[].name}{"\n"}' | xargs -I {} oc -n d8f105-dev set volumes dc/mongodb-samwarren --remove '--name={}'

  # Add a new volume by creating a PVC. If the PVC already exists, omit '--claim-class', '--claim-mode', and '--claim-size' arguments
  oc -n d8f105-dev set volume dc/mongodb-samwarren --add --name=mongodb-samwarren-data -m /var/lib/mongodb/data -t pvc --claim-name=mongodb-samwarren-file --claim-class=netapp-file-standard --claim-mode=ReadWriteMany --claim-size=1G
  ```

- Scale up `mongodb-samwarren` to 1 pods
  ```oc:cli
  oc -n d8f105-dev scale dc/mongodb-samwarren --replicas=1
  ```
- Redeploy with Rolling deployment
  ```oc:cli
  # you can resume rollout; or
  oc -n d8f105-dev rollout resume dc/mongodb-samwarren
  oc -n d8f105-dev rollout latest dc/mongodb-samwarren
  ```

### Fixing it

**Objective**: Fix corrupted MongoDB storage by using the correct storage class for MongoDB.

After using the `RWX` PVC with rolling deployment, you got to a point where your mongodb is now corrupted. That happens because MongoDB does NOT support multiple processes/pods reading/writing to the same location/mount (`/var/lib/mongodb/data`) of single/shared pvc.

To fix that we will need to replace the `RWX` PVC with a `RWO` PVC and change the deployment strategy from `Rolling` to `Recreate` as follow:

- Scale down `rocketchat-samwarren` to 0 pods
  ```oc:cli
  oc -n d8f105-dev scale deployment/rocketchat-samwarren --replicas=0
  ```
- Scale down `mongodb-samwarren` to 0 pods
  ```oc:cli
  oc -n d8f105-dev scale dc/mongodb-samwarren --replicas=0
  ```
- Go to the `mongodb-samwarren` DeploymentConfig and `Pause Rollouts` (under `Actions` menu on the top right side)
  ```oc:cli
    oc -n d8f105-dev rollout pause dc/mongodb-samwarren
  ```
- Remove all existing volumes on `mongodb-samwarren`
  ```oc:cli
  # Remove all volumes
  oc -n d8f105-dev get dc/mongodb-samwarren -o jsonpath='{.spec.template.spec.volumes[].name}{"\n"}' | xargs -I {} oc -n d8f105-dev set volumes dc/mongodb-samwarren --remove '--name={}'
  ```
- Attach a new volume using the existing `mongodb-samwarren-file` PVC
  ```oc:cli
  oc -n d8f105-dev set volume dc/mongodb-samwarren --add --name=mongodb-samwarren-data -m /var/lib/mongodb/data -t pvc --claim-name=mongodb-samwarren-file
  ```
- Change the deployment strategy to use `Recreate` deployment strategy
  ```oc:cli
  oc -n d8f105-dev patch dc/mongodb-samwarren -p '{"spec":{"strategy":{"activeDeadlineSeconds":21600,"recreateParams":{"timeoutSeconds":600},"resources":{},"type":"Recreate"}}}'
  ```
- Go to the `mongodb-samwarren` DeploymentConfig and `Resume Rollouts` (under `Actions` menu on the top right side)
  ```oc:cli
  oc -n d8f105-dev rollout resume dc/mongodb-samwarren
  ```
- Check if a new deployment is being rollout. If not, please do a manual deployment by clicking on `Deploy`
  ```oc:cli
    oc -n d8f105-dev rollout latest dc/mongodb-samwarren
  ```
- Scale up `mongodb-samwarren` to 1 pod, and wait for the pod to become ready
  ```oc:cli
  oc -n d8f105-dev scale dc/mongodb-samwarren --replicas=1
  ```
- Scale up `rocketchat-samwarren` to 1 pod, and wait for the pod to become ready
  ```oc:cli
  oc -n d8f105-dev scale deployment/rocketchat-samwarren --replicas=1
  ```
- Check deployment and make sure `mongodb-samwarren-file-rwx` PVCs are not being used, and delete those PVCs.
  `oc:cli
  oc -n d8f105-dev delete pvc/mongodb-samwarren-file-rwx
  `
  Next page - [Persistent Configurations](./09_persistent_configurations.md)
