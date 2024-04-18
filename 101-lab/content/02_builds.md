# Builds

In this lab, you will create a simple Docker based build for the Rocket Chat application.

<kbd>[![Video Walkthrough Thumbnail](././images/02_builds_thumb.png)](https://youtu.be/j7a74_I6MYw)<kbd>

[Video walkthrough](https://youtu.be/j7a74_I6MYw)

## The Tools Project

The tools project is what will hold various support tools for the application. In this case, all builds will run in this project.

## Creating a Docker-Based Build

The Rocket.Chat application build will be based off a minimal Dockerfile in a [public repository](https://github.com/BCDevOps/devops-platform-workshops-labs/tree/master/apps/rocketchat).
Leveraging the commandline, you can use the `oc new-build` command to create all of the necessary
OpenShift build components.

Ensure that all team members have edit rights into the project. Once complete,
each member can create their own Rocket.Chat docker build.

**Note** In this lab, we'll use square brackets to indicate when you need to replace part of a command and omit the square brackets. If you see `d8f105-tools` in a command, replace that part of the command with the name of your tools namespace. When `d8f105-dev` is indicated, replace this part of the command with your dev namespace's name. In the example below, `oc project d8f105-tools` would become `oc project d8f105-tools` or the name of the tools namespace you are using for this training. We also use samwarren as a unique inditifier throughout the lab, and this can be any unique username so long as you use it consistently throughout the lab.

- To start, switch to the **Tools Project**

```
oc project d8f105-tools
```

- With the `oc` cli, create the build

```oc:cli
oc -n d8f105-tools new-build https://github.com/BCDevOps/devops-platform-workshops-labs/ --context-dir=apps/rocketchat --name=rocketchat-samwarren
```

- The output of the previous command should be similar to the following:

```

--> Found image 8431f8b (21 hours old) in image stream "ocp101a-tools/rocketchat" under tag "latest" for "rocketchat"

    Node.js 8
    ---------
    Node.js 8 available as container is a base platform for building and running various Node.js 8 applications and frameworks. Node.js is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.

    Tags: builder, nodejs, nodejs8

    * A Docker build using source code from https://github.com/BCDevOps/devops-platform-workshops-labs/ will be created
      * The resulting image will be pushed to image stream "rocketchat-samwarren:latest"
      * Use 'start-build' to trigger a new build

--> Creating resources with label build=rocketchat-samwarren ...
    imagestream "rocketchat-samwarren" created
    buildconfig "rocketchat-samwarren" created
--> Success
    Build configuration "rocketchat-samwarren" created and build triggered.
    Run 'oc logs -f bc/rocketchat-samwarren' to stream the build progress.
```

- The build will take between a couple of minutes to about 15 minutes

```oc:cli
# Watch and wait for build
oc -n d8f105-tools logs -f bc/rocketchat-samwarren
```

- You can now explore the Web Console to watch the build status from `Builds`
  _note_ you will see multiple builds from each team member

<kbd>![](./images/01_builds.png)</kbd>

- Or this can be done on the CLI

```
oc -n d8f105-tools get bc
oc -n d8f105-tools status
```

- The build status can be monitored from the Web Console by selecting the `Logs` link

<kbd>![](./images/01_build_logs.png)</kbd>
<kbd>![](./images/01_build_logs_02.png)</kbd>
<kbd>![](./images/01_build_logs_03.png)</kbd>

Next page - [Deployment](./03_deployment.md)
