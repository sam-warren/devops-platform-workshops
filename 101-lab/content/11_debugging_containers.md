# Debugging Containers

### Accessing Local Logs

Logs of a running pod can be accessed from the Web Console or from the `oc` cli:

- The `Logs` tab of any running pod can be used to view active logs for the current pod

<kbd>![](./images/09_debugging_00.png)</kbd>

- The `oc` command can be used to view or tail the logs:

```
oc -n d8f105-dev logs -f <pod name>
```

If there is more than one container in a given pod, the `-c <container-name>` switch is used to specify the desired container logs.

### Using a Debug Container

**Objective**: Create some error on app pod to start debugging:
In this lab, we will scale down the database deployment so that application pods will experience errors and crash.

- Scale down database:
  ```
  oc -n d8f105-dev scale dc/mongodb-samwarren --replicas=0
  ```
- Restart rocketchat:
  ```
  oc -n d8f105-dev rollout restart deployment/rocketchat-samwarren
  ```
- Once the new pod starts, notice the CrashLoopBackOff

<kbd>![](./images/09_debugging_01.png)</kbd>

#### Using the `oc` command to start a debug container

- Find the name of a pod you would like to debug
  ```
  oc -n d8f105-dev get pods
  ```
- Run the `oc debug` command to start a debug pod (your output will vary)
  ```
  $ oc -n d8f105-dev debug [rocketchat-pod-name]
  ```
- Open a new separate terminal window, and view all of the containers in your debug pod using:

  ```
  oc describe pod/[rocketchat-debug-pod-name] -n d8f105-dev
  ```

- After this is complete, return to your debug pod terminal and run the `exit` command. This will remove the debug pod.

  ```
  sh-4.2$ exit
  exit

  Removing debug pod ...
  ```

- Investigate the logs of your rocketchat application pod to get further information about the errors we caused by shutting down the database.

- Resolve the crash loop backoff by scaling your database back to have 1 replica:
  ```
  oc -n d8f105-dev scale dc/mongodb-samwarren --replicas=1
  ```

### RSH and RSYNC

RSH is available to all normal pods through the web console under the `Terminal` tab, as well as through the
`oc rsh` command.

- With your choice of access, rsh into one of the application pods and test access within the namespace
  - cURL internal and external resources
  - Test internal name resolution, external name resolution, etc.
  - Explore your userid

RSYNC is also available in many pods, available through the `oc rsync` command.

- On the CLI, type `oc rsync -h`
- Using this command, copy the contents of the mongo data directory to your local machine, or from your machine to the remote pod

### Port Forwarding

The `oc port-forward` command enables users to forward remote ports running in the cluster
into a local development machine.

- Find your pod and use the port forward command

```
oc -n d8f105-dev get pods  | grep rocketchat-samwarren
oc -n d8f105-dev port-forward [pod name from above] 8000:3000
```

- Navigate to http://127.0.0.1:8000

Next page - [Logging and Visualizations](./12_logging_and_visualizations.md)
