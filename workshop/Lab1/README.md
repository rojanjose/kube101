# Lab 1. Set up and deploy your first application

Learn how to deploy an application to a Kubernetes cluster hosted within
the IBM Container Service.

# 0. Log into the IBM Kubernetes Cluster

1. Log into IBM Cloud.

Use the credentials you got from the lab instrcutor to log into the client VM.

Once logged in, connect to IBM Cloud and your kubernetes clusters as shown in the list of commands below:

```
ibmcloud login
API endpoint: https://cloud.ibm.com

Email> rojanjose@gmail.com

Password> 
Authenticating...
OK

Select an account:
1. Rojan Jose's Account (63431c2ec26fd49d6b5asdfsd32e629c3d)
2. DBG PoC Account - Multiple Clients (bda3b5ea09b5c9cb909067cd839e95c2) <-> 1620097
Enter a number> 2
Targeted account DBG PoC Account - Multiple Clients (bda3b5ea09b5c9cb909067cd839e95c2) <-> 1620097


Select a region (or press enter to skip):
1. au-syd
2. jp-tok
3. eu-de
4. eu-gb
5. us-south
6. us-east
Enter a number> 6
Targeted region us-east

                      
API endpoint:      https://cloud.ibm.com   
Region:            us-east   
User:              rojanjose@gmail.com   
Account:           DBG PoC Account - Multiple Clients (bda3b5ea09b5c9cb909067cd839e95c2) <-> 1620097   
Resource group:    No resource group targeted, use 'ibmcloud target -g RESOURCE_GROUP'   
CF API endpoint:      
Org:                  
Space:                

Tip: If you are managing Cloud Foundry applications and services
- Use 'ibmcloud target --cf' to target Cloud Foundry org/space interactively, or use 'ibmcloud target --cf-api ENDPOINT -o ORG -s SPACE' to target the org/space.
- Use 'ibmcloud cf' if you want to run the Cloud Foundry CLI with current IBM Cloud CLI context.
```

2. Connect and use the cluster:

List the clusters and locate the cluster corresponding to the userId you used to login to the console-in-a-browser environment. For example, if you are user028, your cluster will be user028-cluster.

```
~$ ibmcloud ks clusters

OK
Name              ID                                 State    Created      Workers   Location          Version       Resource Group Name   
user001-cluster   707f162b19cc4c3bbb28bfbfe85ee873   normal   2 days ago   2         Washington D.C.   1.11.8_1547   IKS-RG1   
user026-cluster   dcfb22a45c8e410e8d6f7a8269c2d0c3   normal   1 day ago    2         Washington D.C.   1.11.8_1547   IKS-RG1   
user028-cluster   1b3398b985d84e9b8e9544a91d61428a   normal   1 day ago    2         Washington D.C.   1.11.8_1547   IKS-RG1   
user029-cluster   6d267a184154407d873f0b02159feb84   normal   1 day ago    2         Washington D.C.   1.11.8_1547   IKS-RG1   
```

Configure your Kubernetes client using this command. This will also configure your Kubernetes client for future login sessions by adding the command into your .bash_profile. Note $USER automatically resolves to the user logged into the terminal environment.
```
~$ eval $(ibmcloud ks cluster-config --cluster $USER-cluster --export | tee -a ~/.bash_profile) 
```
You should be able to use kubectl to list kubernetes resources. Try getting the list of pods (there should be none yet)
```
~$ kubectl get pods
No resources found.
```

Once your client is configured, you are ready to deploy your first application, `guestbook`.

# 1. Deploy your application

In this part of the lab we will deploy an application called `guestbook`
that has already been built and uploaded to DockerHub under the name
`ibmcom/guestbook:v1`.

1. Start by running `guestbook`:

   ```$ kubectl run guestbook --image=ibmcom/guestbook:v1```

   This action will take a bit of time. To check the status of the running application,
   you can use `$ kubectl get pods`.

   You should see output similar to the following:

   ```console
   $ kubectl get pods
   NAME                          READY     STATUS              RESTARTS   AGE
   guestbook-59bd679fdc-bxdg7    0/1       ContainerCreating   0          1m
   ```
   Eventually, the status should show up as `Running`.
   
   ```console
   $ kubectl get pods
   NAME                          READY     STATUS              RESTARTS   AGE
   guestbook-59bd679fdc-bxdg7    1/1       Running             0          1m
   ```
   
   The end result of the run command is not just the pod containing our application containers,
   but a Deployment resource that manages the lifecycle of those pods.
 
   
3. Once the status reads `Running`, we need to expose that deployment as a
   service so we can access it through the IP of the worker nodes.
   The `guestbook` application listens on port 3000.  Run:

   ```console
   $ kubectl expose deployment guestbook --type="NodePort" --port=3000
   service "guestbook" exposed
   ```

4. To find the port used on that worker node, examine your new service:

   ```console
   $ kubectl get service guestbook
   NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
   guestbook   NodePort   10.10.10.253   <none>        3000:31208/TCP   1m
   ```
   
   We can see that our `<nodeport>` is `31208`. We can see in the output the port mapping from 3000 inside 
   the pod exposed to the cluster on port 31208. This port in the 31000 range is automatically chosen, 
   and could be different for you.

5. `guestbook` is now running on your cluster, and exposed to the internet. We need to find out where it is accessible.
   The worker nodes running in the container service get external IP addresses.
   Run `$ ibmcloud cs workers <name-of-cluster>`, and note the public IP listed on the `<public-IP>` line.
   
   ```console
   $ ibmcloud cs workers osscluster
   OK
   ID                                                 Public IP        Private IP     Machine Type   State    Status   Zone    Version  
   kube-hou02-pa1e3ee39f549640aebea69a444f51fe55-w1   173.193.99.136   10.76.194.30   free           normal   Ready    hou02   1.5.6_1500*
   ```
   
   We can see that our `<public-IP>` is `173.193.99.136`.
   
6. Now that you have both the address and the port, you can now access the application in the web browser
   at `<public-IP>:<nodeport>`. In the example case this is `173.193.99.136:31208`.
   
Congratulations, you've now deployed an application to Kubernetes!

When you're all done, you can either use this deployment in the
[next lab of this course](../Lab2/README.md), or you can remove the deployment
and thus stop taking the course.

  1. To remove the deployment, use `$ kubectl delete deployment guestbook`.

  2. To remove the service, use `$ kubectl delete service guestbook`.

You should now go back up to the root of the repository in preparation
for the next lab: `$ cd ..`.
