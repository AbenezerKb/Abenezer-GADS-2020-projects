#LAB: GCP Fundamentals: Getting Started with Kubernetes Engine

## Objectives:

In this lab, you will learn how to perform the following tasks
    * Provision a Kubernetes cluster using Kubernetes Engine
    * Package a simple ASP.NET Core app as a Docker container
    * Deploy and manage Docker containers using kubectl
    * Deploy your ASP.NET Core app to a pod
    * Scale up your service and roll out an upgrade

##Steps

1. Start a Kubernetes Engine cluster

    - place the zone from qwicklabs to variable called MY_ZONE:

        export MY_ZONE=us-central1-a
    
    - start kubernetes cluster managed by kubernetes engine, with cluster name hello-dotnet-cluster and 2 nodes. This command will authenticate kubectl:

        gcloud container clusters create hello-dotnet-cluster --zone $MY_ZONE --num-nodes 2

    - Output:
        Creating cluster hello-dotnet-cluster...done.
        Created [https://container.googleapis.com/v1/projects/dotnet-atamel/zones/europe-west1-b/clusters/hello-dotnet-cluster].
        kubeconfig entry generated for hello-dotnet-cluster.
        NAME                  LOCATION          MASTER_VERSION  MASTER_IP      MACHINE_TYPE    NODE_VERSION   NUM_NODES  STATUS
        hello-dotnet-cluster  us-central1-a  1.14.10-gke.27   35.195.57.183  n1-standard-1  1.14.10-gke.27  2          RUNNING

    - After the cluster is created, check your installed version of Kubernetes:

        kubectl version
    
2. Create an ASP.NET Core app in Cloud Shell

    - Verify dotnet version:

        dotnet --version

      Output: 3.1.201

    - Disable telemetry coming from the new app:

        export DOTNET_CLI_TELEMETRY_OPTOUT=1

    - Create a skeleton ASP.NET Core web app using the following dotnet command:

        dotnet new razor -o HelloWorldAspNetCore

      Output: 
      Restore completed in 11.44 sec for HelloWorldAspNetCore.csproj.
      Restore succeeded.  
3. Run the ASP.NET Core app

    - Navigate to project folder:

        cd HelloWorldAspNetCore

    - Run the app

        dotnet run --urls=http://localhost:8080

      Output: 

        Hosting environment: Production
        Content root path: /home/atameldev/HelloWorldAspNetCore
        Now listening on: http://[::]:8080
        Application started. Press Ctrl+C to shut down.   

4. Publish the ASP.NET Core app

    -  Publish the app to get a self-contained DLL using the dotnet publish command:

        dotnet publish -c Release

      Output: 
      HelloWorldAspNetCore -> /home/gcpstaging55636_student/HelloWorldAspNetCore/bin/   Release/netcoreapp3.1/HelloWorldAspNetCore.dll 

    - Navigate to the the publish folder

        cd bin/Release/netcoreapp3.1/publish/

5. Package the ASP.NET Core app as a Docker container

    - Create a Dockerfile to define the Docker image:

        touch Dockerfile

    - Add the following to Dockerfile using editor and save it:

        FROM gcr.io/google-appengine/aspnetcore:3.1 ADD ./ /app
        ENV ASPNETCORE_URLS=http://*:${PORT}
        WORKDIR /app
        ENTRYPOINT [ "dotnet", "HelloWorldAspNetCore.dll" ]

    - Build an image, replacing <project_id> with your Google Cloud Project ID:

        DEVSHELL_PROJECT_ID=<project_id> docker build -t gcr.io/$DEVSHELL_PROJECT_ID/hello-dotnet:v1 .

    - Test the image locally with the following command, which runs a Docker container as a daemon on port 8080 from your newly created container image:

        docker run -d -p 8080:8080 gcr.io/$DEVSHELL_PROJECT_ID/hello-dotnet:v1

    - After you verify that the app is running locally in a Docker container, you can stop the running container.   First, get the container ID:

        docker ps

    - Stop the container, replacing <Container_ID> with your ID number:

        docker stop <Container_ID>    

      Output: 

      CONTAINER ID         IMAGE                              COMMAND
    ced2872b26fc        gcr.io/PROJECT_ID/hello-dotnet:v1    "dotnet HelloWorld
    $ docker stop ced2872b26fc
    ced2872b26fc    

    - Set the project with this command:

        gcloud config set project $DEVSHELL_PROJECT_ID

    - Now that the image works as intended, you can push it to the Google Container Registry, a private repository for your Docker images accessible from every Google Cloud project (but also from outside Google Cloud Platform) :

        gcloud docker -- push gcr.io/$DEVSHELL_PROJECT_ID/hello-dotnet:v1

    - To see the list of images in container

        gcloud container images list

6. Create and deploy your pod to cluster

    - Create a pod with the kubectl create command:

        kubectl create deployment hello-dotnet --image=gcr.io/$DEVSHELL_PROJECT_ID/hello-dotnet:v1

    - To view the deployment you just created, run:

        kubectl get deployments

      Output: 
              NAME           READY    UP-TO-DATE   AVAILABLE   AGE
              hello-dotnet   1/1         1          1          37s
    - To view the pod created by the deployment, run this command:

        kubectl get pods

      Output: NAME                         READY     STATUS    RESTARTS   AGE
              hello-dotnet-714049816-ztzrb   1/1       Running   0        57s  
    
    - Expose the hello-dotnet container to the internet:

        kubectl expose deployment hello-dotnet --type="LoadBalancer" --port=8080

    - View the new service:

        kubectl get services

    - Open a new web browser tab and go to

        http://<external IP address>:8080

    - Scale up the number of pods running on your service:

        kubectl scale deployment hello-dotnet --replicas 3

    - Confirm that Kubernetes has updated the number of pods:

        kubectl get pods

    - Confirm that your external IP address has not changed:

        kubectl get services