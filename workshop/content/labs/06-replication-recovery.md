### Replication and Recovery

#### Things will go wrong, that's why we have replication and recovery

Things will go wrong with your software, or your hardware, or from something out of your control. But we can plan for that failure, and planning for it let's us minimize the impact. OpenShift supports this via what we call replication and recovery.

##### Replication

Let's walk through a simple example of how the replication controller can keep your deployment at a desired state. 

##### *CLI Instructions (Option 1)*

Try the following command in the terminal:

```execute
oc scale --replicas=4 dc/demojam
```

Check out the new pods: 

```execute
oc get pods
```

Notice that you now have 4 unique pods availble to inspect. If you want go ahead and inspect them, using ```oc describe pod [POD NAME]```, you can see that each have their own IP address and logs.

##### *Web Console Instructions (Option 2)*

From the left navbar, navigate to ```Home``` > [Status](%console_url%/overview/ns/demo-%username%). Click on your ```(DC)  demojam``` resource. On the right panel popup, click on ```Overview```:

>(DC) is short for deployment configuration.

![dj_overview](images/lab6_workshop_dj_overview.png)

Underneath overview, click ```DESIRED COUNT``` and set it to 4. The deployment should indicate that it is scaling to 4 pods, and eventually you will have 4 running pods. Keep in mind that each pod has it's own container which is an identical deployment of the webapp. OpenShift is now (by default) round robin load-balancing traffic to each pod.

![dj_scale_4](images/lab6_workshop_dj_scale_4.png)

So you've told OpenShift that you'd like to maintain 4 running, load-balanced, instances of our web app.

#### Recovery

Okay, now that we have a slightly more interesting replication state, we can test a service outages scenario. In this scenario, the dc-metro-map replication controller will ensure that other pods are created to replace those that become unhealthy. Let's force inflict an issue and see how OpenShift reponds.

##### *CLI Instructions (Option 1)*

Choose a random pod and delete it:

```execute
oc get pods
```

```
oc delete pod/PODNAME
```

```execute
oc get pods -w
```

If you're fast enough you'll see the pod you deleted go ```Terminating``` and you'll also see a new pod immediately get created and transition from ```Pending``` to ```Running```. If you weren't fast enough you can see that your old pod is gone and a new pod is in the list with an age of only a few seconds.

You can see the more details about your replication controller with:

```execute
oc describe rc
```

<br>

##### *Web Console Instructions (Option 2)*

On the left navbar, click on ```Workload``` > [Deployment Config](%console_url%/k8s/ns/demo-%username%/deploymentconfigs). Click on your pod. 

Then, select the ```Pods``` tab in between the ```YAML``` and ```Environment``` tabs. You should see your 4 pods: 

![dj_resources_4](images/lab6_workshop_dj_resources_4.png)

On the right side, for any pod, click the vertical ellipses ```...``` button, and select ```Delete Pod```:

![dj_pod_delete](images/lab6_workshop_dj_pod_delete.png)

Confirm deletion, you'll see that pod enter ```Terminating``` status and another pod appear with ```Container Creating``` status: 

![dj_pod_restart](images/lab6_workshop_dj_delete_restart.png)

<br>

#### Application Health

In addition to the health of your application's pods, OpenShift will watch the containers inside those pods. Let's force inflict some issues and see how OpenShift reponds.

##### *CLI Instructions (Option 1)*

Choose a running pod and shell into it:

```execute
oc get pods
```

```
oc exec PODNAME -it /bin/bash
```

You are now executing a bash shell running in the container of the pod. Let's kill our webapp and see what happens.

> If we had multiple containers in the pod we could use the ```-c CONTAINER_NAME``` flag to select the appropriate one.

```
pkill -9 node
```

This will kick you out off the container with an error like: 

```
command terminated with exit code 137
```

Repeat the process: Shell in and execute the same command to kill node. 

After executing the pkill command, attempt to shell in again, and you should see this message instead:

```
error: unable to upgrade connection: container not found
```

Watch for the container restart
```execute
oc get pods -w
```

If a container dies multiple times quickly, OpenShift is going to put the pod in a ```CrashBackOff``` state. This ensures the system doesn't waste resources trying to restart containers that are continuously crashing.

<br>

##### *Web Console Instructions (Option 2)*

On the left navbar, click on ```Workload``` > [Pods](%console_url%/k8s/ns/demo-%username%/pods). Select any running pod. 

Click the ```Terminal``` next to the ```Events``` tab:

![dj_pod_terminal](images/lab6_workshop_dj_pod_terminal.png)

Click inside the terminal view and type 

```
pkill -9 node
```

![dj_terminal_pkill](images/lab6_workshop_pkill.png)

This is going to kill the node.js web server and kick you off the container.

![dj_pod_closed](images/lab6_workshop_dj_term_cli_closed.png)

In the top right of the workshop, click the ```Settings Icon``` > ```Reload Console```:

![reload_console](images/lab6_workshop_reload_console.png)

Reload the console multiple times.

Go back to the pods list.

The container died multiple times so quickly that OpenShift is going to put the pod in a ```CrashBackOff``` state. This ensures the system doesn't waste resources trying to restart containers that are continuously crashing.

<br>

#### Clean up

Let's scale back down to 1 replica. 

If you are using the command line use the ```oc scale``` command. 

```execute
oc scale --replicas=1 dc/demojam
```

From the dashboard, you can just edit the ```DESIRED COUNT``` in the ```Status``` > ```(DC) demojam``` > ```Overview``` to 1: 

![dj_overview](images/lab6_workshop_dj_overview.png)

<br>

#### Summary

In this lab we learned about replication controllers and how they can be used to scale your applications and services. We also tried to break a few things and saw how OpenShift responded to heal the system and keep it running. This topic can get deeper than we've experimented with here, but getting deeper into application health and recovery is an advanced topic.