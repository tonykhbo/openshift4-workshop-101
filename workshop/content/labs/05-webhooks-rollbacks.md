### Build Triggers, Webhooks and Rollbacks

Once you have an app deployed in OpenShift you can take advantage of some continuous capabilities that help to enable DevOps and automate your management process. We will cover some of those in this lab: Build triggers, webhooks, and rollbacks.

#### Using Github
We are going to do some integration and coding with an external git repository. For this lab we are going to use github, if you don't already have an account, you can create one here, https://github.com/join

OK, let's fork the dc-metro-map app from my account into your github account. Goto https://github.com/tonykhbo/demojam and look to the top right for the "Fork" button.

![githubfork](images/lab5_workshop_github_fork.png)

Click the "Fork" button and Github should redirect you to the newly created fork of the source code.

#### Build Trigger / Code Change Webhook

When using S2I there are a few different things that can be used to trigger a rebuild of your source code. The first is a configuration change, the second is an image change, and the last (which we are covering here) is a webhook. A webhook is basically your git source code repository telling OpenShift that the code we care about has changed. Let's set that up for our project now to see it in action.

Jump back to your OpenShift web console and let's add the webapp to our project. You should know how to do this from previous lab work, but this time point to your github URL for the source code. If you need a refresher expand the box below.

##### Web Console Instructions

In the Developer View, click on +Add, then select "From Git" in the available options.

Enter the Git Url from the forked repository in YOUR account 

It should look something like this (without the brackets:

https://github.com/[YOUR_ACCOUNT]/demojam.git

Select NodeJS, Version 10

![personalgit](images/lab5_workshop_new_git.png)

Scroll down and set the Application Name and Name to demojam. Click "Create": 

![gitappname](images/lab5_workshop_appname.png)

#### Grabbing the Webhooks
The node.js builder template creates a number of resources for you, but what we care about right now is the build configuration because that contains the webhooks. 

<br>

##### *CLI Instructions (Option 1)*

Inside the terminal, run the following command: 

```execute
oc describe bc/demojam | grep -i webhook
```

##### *Web Console Instructions (Option 2)*

Jump back to the Admin View, Expands the Builds tab on the left hand side, then click on "Build Config":

![djbuildc](images/lab5_workshop_demojam-bc.png)

Scroll down to find the Web Hooks section. Click "Copy URL with Secret" for the GENERIC WebHook

![genericwh](images/lab5_workshop_generic_webhook.png)

#### Jump back over to Github

After copying down the Webhook URL from the CLI Instructions or Web Console instructions, navigate back to your forked repository on github. 

![githubsettings](images/lab5_workshop_github_settings.png)

Click on the "Settings" Tab, then click on "Webhooks" on the left hand pane. 

![ghwebhook](images/lab5_workshop_githubwebhooks.png)

Click the "Add Webhook" button and paste in the generic webhook url you copied. Click "Disable SSL Verification" and leave the rest of the settings as default. Click "Add Webhook" button: 

![ghwhadd](images/lab5_workshop_githubwh_add.png)

Good work! Now any "push" to the forked repository will send a webhook that triggers OpenShift to: re-build the code and image using s2i, and then perform a new pod deployment. In fact Github should have sent a test trigger and OpenShift should have kicked off a new build already.

##### Deployment Triggers

In addition to setting up triggers for rebuilding code, we can setup a different type of trigger to deploy pods. Deployment triggers can be due to a configuration change (e.g. environment variables) or due to an image change. This powerful feature will be covered in one of the advanced labs. See the Triggers link under More Information below.

#### Rollbacks

Well, what if something isn't quite right with the latest version of our app? Let's say some feature we thought was ready for the world really isn't - and we didn't figure that out until after we deployed it. No problem, we can roll it back with the click of a button. 

##### *CLI Instructions (Option 1)*

In the terminal, run the following command:

```execute
oc rollback demojam-1
```


Check the status of your pods:

```execute
oc get pods -w
```

OpenShift has done a graceful removal of the old pod and created a new one.

> Note that the old pod wasn't killed until the new pod was successfully started and ready to be used. This is so that OpenShift could continue to route traffic to the old pod until the new one was ready.

>You can integrate your CI/CD tools to do rollbacks with the REST API. See the Rollbacks With the REST API link under More Information below.

<br>

#### Summary

In this lab we saw how you can configure a source code repository to trigger builds with webhooks. This webhook could come from Github, Jenkins, Travis-CI, or any tool capable of sending a URL POST. Keep in mind that there are other types of build triggers you can setup. For example: if a new version of the upstream RHEL image changes. We also inspected our deployment history and did a rollback of our running deployment to one based on an older image with the click of a button.

##### More information

[Triggers](https://docs.openshift.com/container-platform/4.2/builds/triggering-builds-build-hooks.html)

[Deployment Configs and Rollbacks](https://docs.openshift.com/container-platform/4.2/applications/deployments/what-deployments-are.html)


