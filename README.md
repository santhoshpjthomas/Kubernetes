**# Kubernetes**

Testing Kubernetes with my first deployment

**1.Overview:**

Kubernetes is an open source project which can run in many different environments, from laptops to high-availability multi-node clusters, from public clouds to on-premise deployments, from virtual machines to bare metal.

What I have here is deploy a simple java application into Kubernetes running on Container Engine.

For the purpose of deploying a simple Java application, we use a managed environment such as Container Engine (a Google-hosted version of Kubernetes running on Compute Engine) allows you to focus more on experiencing Kubernetes rather than setting up the underlying infrastructure.

**2. Setup and Requirements:**

- Create a Google Cloud account if you don&#39;t have one and create a New Project.
- Java is already installed in Cloud Shell.
- Run sudo update-alternatives --config javac to check Java 8 is running.
- Run sudo update-alternatives --config java and check Java 8 is running.

**3. Get the example Spring Boot code:**

          Get the sample code from the following path:

           https://github.com/santhoshpjthomas/Kubernetes.git

**4. Run the application locally:**

       After cloning the code to the project test the application by running it locally using the following command. After the application is started select the Web-Preview option which will run the application in browser.

             ./mvnw -DskipTests spring-boot:run

    P.S: Start the spring application using Spring Boot Plugin

**5. Package the application as Docker Container:**

**       Prepare the application to run on Kubernetes. As a first step make the application as a Docker image.Create the JAR deployable for the application.

              ./mvnw -DskipTests package

        Create a DockerFile

               touch Dockerfile

         Add the following to Dockerfile using favourite editor

              FROM openjdk:8

              COPY target/gs-spring-boot-0.1.0.jar /app.jar

              EXPOSE 8080/tcp

              ENTRYPOINT [&quot;java&quot;, &quot;-jar&quot;, &quot;/app.jar&quot;]

        The Dockerfile shown above builds on the OpenJDK image, which is already configured to have OpenJDK pre-installed and can run your JAR.

         Save this Dockerfile and build this image by running this command (Make sure to replace the Project-Id with yours)

              docker build -t gcr.io/PROJECT\_ID/hello-java:v1 .

       Once this completes (it&#39;ll take some time to download and extract everything) you can test the image locally with the following command which will run a Docker container as a daemon on port 8080 from your newly-created container image:

                 docker run -ti --rm -p 8080:8080 gcr.io/PROJECT\_ID/hello-java:v1

        You should see the default page in a new tab. Once you verify that the app is running fine locally in a Docker container, you can stop the running container by pressing Ctrl+C

         Now that the image works as intended you can push it to the Google Container Registry, a private repository for your Docker images accessible from every Google Cloud project (but also from outside Google Cloud Platform) :

                  gcloud docker -- push gcr.io/PROJECT\_ID/hello-java:v1

         If all goes well and after a little while you should be able to see the container image listed in the console: Compute &gt; Container Engine &gt; Container Registry. At this point you now have a project-wide Docker image available which Kubernetes can access and orchestrate as you&#39;ll see in a few minutes.

**6. Create Cluster:**

**       ** A cluster consists of a Kubernetes master API server managed by Google and a set of worker nodes. The worker nodes are Compute Engine virtual machines. Let&#39;s use the gcloud CLI from your CloudShell session to create a cluster with two n1-standard-1 nodes (this will take a few minutes to complete)

                   gcloud container clusters create hello-java-cluster \

                  --num-nodes 2 \

                  --machine-type n1-standard-1 \

                  --zone us-central1-c

    After this a cluster will be created and you will be able to view in Google Container Engine.

**7. Deploy application to Kubernetes:**

**         ** A Kubernetes deployment can create, manage, and scale multiple instances of your application using the container image you&#39;ve just created. Let&#39;s deploy one instance of your application into Kubernetes using the kubectl run command.

             kubectl run hello-java \

             --image=gcr.io/PROJECT\_ID/hello-java:v1 \

             --port=8080

To view the deployment which has just been create, use the following command.

             kubectl get deployments

To view application instances created by this deployment us the following command.

                kubectl get pods

At this point you should have your container running under the control of Kubernetes but you still have to make it accessible to the outside world.

**8. External Traffic:**

**     ** By default, the pod is only accessible by its internal IP within the cluster. In order to make the hello-java container accessible from outside the kubernetes virtual network, you have to expose the pod as a kubernetes service.

       Using Cloud Shell expose the pod to the public internet with the kubectl expose command combined with the --type=LoadBalancer flag. This flag is required for the creation of an externally accessible IP :

       kubectl expose deployment hello-java --type=LoadBalancer

      Note that you expose the deployment, and not the pod directly. This will cause the resulting service to load balance traffic across all pods managed by the deployment.

     The Kubernetes master creates the load balancer and related Compute Engine forwarding rules, target pools, and firewall rules to make the service fully accessible from outside of Google Cloud Platform.

       Kubectl get services command will list all the services in the cluster.

You should now be able to reach the service by pointing your browser to this address: http://&lt;EXTERNAL\_IP&gt;:8080

**9. Scale up Service:**

**     ** Kubernetes can scale up your application very easily. Suppose you suddenly need more capacity for your application; you can simply tell the replication controller to manage a new number of replicas for your application instances:

       kubectl scale deployment hello-java --replicas=3

hello-java deployment will be scaled. And enter kubectl get deployment will show the hello-java deployment has been scaled up.

**10. Upgrade service:**

**       ** At some point the application that you&#39;ve deployed to production will require bug fixes or additional features. Kubernetes is here to help you deploy a new version to production without impacting your users

     Rebuild the application with latest changes using the following command.

       ./mvnw -DskipTests package

     After the build the container image using the below command.

       docker build -t gcr.io/PROJECT\_ID/hello-java:v2 .

     Push image into the container registry.

        gcloud docker -- push gcr.io/PROJECT\_ID/hello-java:v2

You can use kubectl set image command to ask Kubernetes to deploy the new version of your application across the entire cluster one instance at a time with rolling update:

       kubectl set image deployment/hello-java \

       hello-java=gcr.io/PROJECT\_ID/hello-java:v2

**11 .Rollback:**

         To make a rollback of the deployment made use the following command,

         kubectl rollout undo deployment/hello-java
