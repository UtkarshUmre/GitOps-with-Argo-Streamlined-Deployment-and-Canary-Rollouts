# GitOps : Deploying a Basic Web Application to Kubernetes via Argo CD, with Canary Release Managed by Argo Rollouts

   <image src="https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExeG4wNTVobzFzeHRwMGd2cjN2eWcyNm9tZGJ2bGJraWFjMmJsYnJraiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/cF95iWl8uhdoMCr08C/giphy.gif">

```This project utilizes GitOps principles for deploying and managing a canary release of a web application using Argo Rollouts. The web application is deployed to a Kubernetes cluster using Argo CD.

In the GitOps ideology, Git serves as the single source of truth. Any changes made to the deployment YAML file are automatically synchronized to achieve the desired state, a process facilitated by Argo CD.

The project also involves a canary release of the web application, which is available in two versions. Argo Rollouts is used to manage this canary release.

For those unfamiliar with the term, a canary release is a gradual, step-by-step release process for an application. Initially, a small portion of traffic is directed towards the new or testing version of the application. For instance, 20% of users may first access the new version, followed by 50%. If the application does not perform as expected, we can roll back to the previous version. If the application performs well, we can roll it out to 100% of users.

This project successfully implements the concept of a canary release with the help of Argo Rollouts.

The following sections provide a detailed, step-by-step procedure of this project, including the challenges faced during its development and how they were resolved. By following this guide, you too can replicate this project. Here is a summary of the project.
```

## step 1

In the initial phase of this project, I set up a local Kubernetes cluster using Minikube, a tool that's primarily used for testing purposes as it allows the creation of a Kubernetes cluster on a local machine.

Once the Kubernetes cluster was up and running, the next step was to install Argo CD, a critical component for this GitOps project. The installation process was carried out by following the official [Argo CD documentation.](https://argo-cd.readthedocs.io/en/stable/getting_started/)

The first step in the installation process was to create a new namespace named 'argocd' in the Kubernetes cluster. This was followed by running a command to apply the Argo CD 'install.yaml' file. This command installs Argo CD services and application resources into the 'argocd' namespace.

With the successful execution of these steps, Argo CD was successfully installed on the Kubernetes cluster, marking the completion of the project initialization phase."

```bash
  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## step 2

In the second step, after successfully installing Argo CD on the Kubernetes cluster, we need to access the Argo CD web interface. This interface provides a visual representation of our deployments and a GUI for interaction, making it a convenient tool for managing our applications.

Since Argo CD is installed within our Kubernetes cluster, and our cluster is running on Minikube, we need to use port forwarding to access the Argo CD web interface from our local machine. Minikube is a lightweight Kubernetes implementation used for testing, and it has some restrictions compared to a full-fledged Kubernetes cluster.

To set up port forwarding, use the following command:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

This command will make the Argo CD web interface accessible at `localhost:8080` on your local machine.

The default username for Argo CD is 'admin'. To retrieve the password for the 'admin' user, use the following command:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

This command retrieves the password for the initial admin user in Argo CD, decodes it from base64, and prints it to the terminal. You can then use this password to log in to the Argo CD web interface."

## step 3

Now, for the deployment on the Kubernetes cluster, I have already created the deployment.yaml and service.yaml files, which can be accessed in the manifests folder in the source code repository. Initially, I used version one of the simple web app in the deployment.yaml file and created the service.yaml file to expose the web app. To create an application in Argo CD, I created the application.yaml file.

This YAML file is used to define an Argo CD Application. Once this file is applied using kubectl apply -f <filename.yaml>, Argo CD will ensure that the state of the Kubernetes cluster matches the state defined in the Git repository. Any changes made in the Git repository will be automatically reflected in the cluster.

<!-- <a href="https://ibb.co/sqXm8Vh"><img src="https://i.ibb.co/L1skVZM/bigsize-comparison.png" alt="bigsize-comparison" border="0"></a> -->

<a href="https://ibb.co/sqXm8Vh">
  <img src="https://i.ibb.co/L1skVZM/bigsize-comparison.png" alt="bigsize-comparison" border="0" width="900">
</a>

---

In this video, I've shown the process of deploying an application to Kubernetes using Argo CD, triggering Argo CD to synchronize changes, and making adjustments in the deployment file. This action then initiated the Argo workflow, ensuring synchronization between the desired and actual states.

---

[Screencast from 14-04-24 08:14:36 AM IST.webm](https://github.com/UtkarshUmre/vodeo-test/assets/112888849/76dea11d-b1e0-476f-87ae-547f79ed749c)

---

So, that was a basic demonstration of utilizing Argo CD for GitOps. In essence, Argo CD streamlines, secures, and improves the application deployment process within a Kubernetes environment.

---

## step 4

To implement the canary release for the application, I utilized Argo Rollouts. Argo Rollouts is a Kubernetes controller and a set of Custom Resource Definitions (CRDs) that offers advanced deployment capabilities such as blue-green, canary, and AB testing for Kubernetes. It extends the Kubernetes API and seamlessly integrates with Kubernetes native services. As part of the Argo Project, a collection of open-source tools for Kubernetes that manages workflows, deployments, and CI/CD processes, Argo Rollouts enhances Kubernetes deployments by providing advanced strategies, easy promotion and rollback, integration with metrics and ingress controllers/service meshes, a Web UI, and CLI for management. To install Argo Rollouts in your Kubernetes cluster, you can execute the following commands:

```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

After installation, you can define a Rollout, which is a custom resource similar to a Deployment. For example, you can store your rollout files in the argoRollout folder, where rollout.yaml and service.yaml reside. In this example, the new version of the application is gradually rolled out. Initially, 20% of the traffic is directed to the new version, followed by 40%, 60%, and finally 100%. Each step is accompanied by a pause to monitor the application's performance. Applying the rollout with kubectl apply -f <filename.yaml> will prompt Argo Rollouts to manage the gradual rollout of the new version according to the steps defined in the Rollout.

To create the rollout and service, you can apply these files using the command kubectl apply -f <file_name>. To monitor rollouts via CLI, use

```bash
kubectl argo rollouts get rollout rollouts-demo --watch
```

and to view them via the web UI, use

```bash
kubectl argo rollouts dashboard
```

To rollout a new image, simply apply the new image in the rollout.yaml file, which serves as a deployment file, using the command

```bash
kubectl argo rollouts set image <rollout_name> <rollout_name>=<image/version>
```

Once the image is set, you can use the aforementioned methods to rollout.

For connecting to the application, the following command works in Minikube; otherwise, you'll need to set up an ingress:

```bash
minikube service --all -n default
```

This is how we can successfully implement a canary release using Argo Rollouts. You can also refer to the video where I have shown a hands-on demo of implementing Argo Rollouts and a canary release. This is how the canary release revisions work when we trigger the rollout.

---

[Screencast from 14-04-24 06:23:46 AM IST.webm](https://github.com/UtkarshUmre/video-test/assets/112888849/c490987e-f339-4498-9b7a-76c9bfa7b95c)

---

### how to cleanly remove all resources created during this assignment from the Kubernetes cluster

To cleanly remove all resources created during the assignment from the Kubernetes cluster, you can follow these steps:

#### 1 delete the namespaces where the resources were created:

commands to delete these namespaces:

```bash
kubectl delete namespace argocd
kubectl delete namespace argo-rollouts
```

deleting a namespace will delete all the resources in that namespace.

If you have created resources in other namespaces or in the default namespace, you would need to delete those as well. If you know the kind and name of the resources, you can delete them individually like so:

```bash
kubectl delete <kind> <name>
```

#### 2 Uninstall Argo CD:

If you installed Argo CD using Helm or any other package manager, use the corresponding command to uninstall it. For example:

```bash
helm uninstall argocd
```

#### 3 Delete Argo Rollouts Resources:

Delete the Argo Rollouts resources, including Rollouts and Services, using kubectl delete command. For example:

```bash
kubectl delete -f rollout.yaml
kubectl delete -f service.yaml
```

#### 4 Uninstall Argo Rollouts:

If you installed Argo Rollouts using Helm or any other package manager, use the corresponding command to uninstall it. For example:

```bash
helm uninstall argo-rollouts
```

#### 5 Clean Up Other Resources:

Delete any other resources that were created specifically for this assignment, such as ConfigMaps, Secrets, or PersistentVolumes.

#### 6 Verify Cleanup:

Use kubectl get commands to verify that all resources have been deleted successfully. For example:

```bash
kubectl get pods
kubectl get services
kubectl get deployments
```

By following these steps, you can cleanly remove all resources created during the assignment from the Kubernetes cluster, ensuring that no unnecessary resources are left behind.

---

## conclusion

In conclusion, this project has been an enlightening journey into the world of GitOps, Kubernetes, Argo CD, and Argo Rollouts. It has demonstrated the power of these tools and methodologies in managing complex deployments in a controlled, versioned, and automated manner.

The use of Argo CD has highlighted the benefits of GitOps, where Git serves as the single source of truth. This approach ensures that the state of the Kubernetes cluster always matches the desired state specified in the Git repository, thereby reducing the risk of configuration drift and enhancing the reproducibility of deployments.

The implementation of a canary release strategy using Argo Rollouts has provided valuable insights into how updates can be rolled out gradually and safely. This strategy reduces the risk of introducing breaking changes to all users at once and allows for quick rollback if any issues are detected.

What I Learned:

This project has been a significant learning experience. I have gained a deep understanding of GitOps principles and how they can be applied to manage deployments on a Kubernetes cluster. I have learned how to use Argo CD for continuous delivery and how to leverage its features to maintain a declarative and version-controlled approach to deployments.

I have also learned how to use Argo Rollouts to manage the release of updates to a web application. I have gained practical experience in implementing a canary release strategy, which has taught me the importance of controlled and gradual rollouts in reducing the risk of breaking changes.

Overall, this project has significantly enhanced my skills and knowledge in GitOps, Kubernetes, Argo CD, and Argo Rollouts, and I am confident that these skills will be highly valuable in my future endeavors.
