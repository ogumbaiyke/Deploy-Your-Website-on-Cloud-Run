# Deploy-Your-Website-on-Cloud-Run

Running websites can be difficult with creating and managing VMs, clusters, pods, services, etc. This is fine for larger, multi-tiered applications, but if you are just trying to get your website deployed and visible, it's a lot of overhead.


With Cloud Run, Google Cloud's implementation of Google's Knative framework, you can manage and deploy your website without any of the infrastructure overhead you experience with a VM or pure Kubernetes-based deployments. Not only is this a simpler approach from a management perspective, it also gives you the ability to "scale to zero" when there are no requests coming into your website.

Cloud Run brings "serverless" development to containers and can be run either on your own Google Kubernetes Engine (GKE) clusters or on a fully managed PaaS solution provided by Cloud Run. You will be running the latter scenario in this lab.

The exercises are ordered to reflect a common cloud developer experience:

Create a Docker container from your application
Deploy the container to Cloud Run
Modify the website
Roll out a new version with zero downtime
What you'll learn
In this lab you will learn how to:

Build a Docker image using Cloud Build and upload it to Artifact Registry
Deploy Docker images to Cloud Run
Manage Cloud Run deployments
Set up an endpoint for an application on Cloud Run






Activate Cloud Shell
Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Cloud Shell provides command-line access to your Google Cloud resources.

Click Activate Cloud Shell Activate Cloud Shell icon at the top of the Google Cloud console.

Click through the following windows:

Continue through the Cloud Shell information window.
Authorize Cloud Shell to use your credentials to make Google Cloud API calls.
When you are connected, you are already authenticated, and the project is set to your Project_ID, qwiklabs-gcp-02-fdbd765f57e6. The output contains a line that declares the Project_ID for this session:

Your Cloud Platform project in this session is set to qwiklabs-gcp-02-fdbd765f57e6
gcloud is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.

(Optional) You can list the active account name with this command:
gcloud auth list
Copied!
Click Authorize.
Output:

ACTIVE: *
ACCOUNT: student-04-5f7e7e1fd578@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`
(Optional) You can list the project ID with this command:
gcloud config list project
Copied!
Output:

[core]
project = qwiklabs-gcp-02-fdbd765f57e6
Note: For full documentation of gcloud, in Google Cloud, refer to the gcloud CLI overview guide.
Task 1. Clone the source repository
Since you are deploying an existing website, you just need to clone the source, so you can focus on creating Docker images and deploying to Cloud Run.

In Cloud Shell run the following commands to clone the git repository and change to the appropriate directory:
git clone https://github.com/googlecodelabs/monolith-to-microservices.git
cd ~/monolith-to-microservices
Copied!
Install the NodeJS dependencies so you can test the application before deploying:
./setup.sh
Copied!
This will take a few minutes to run. You will see a success message when it finishes.

Test your application by running the following command to start the web server:
cd ~/monolith-to-microservices/monolith
npm start
Copied!
Output:

Monolith listening on port 8080!
Preview your application by clicking the web preview icon and selecting Preview on port 8080.
Preview on port 8080 option selected on the expanded web preview menu

This should open a new window where you can see your Fancy Store web page in action.

Fancy Store website

Close this window after viewing the website, and stop the web server process by pressing CTRL+C in Cloud Shell.
Task 2. Create a Docker container with Cloud Build
Now that you have the source files ready to go, it is time to Dockerize your application!

Normally you would have to take a two step approach that entails building a docker container and pushing it to a registry to store the image for GKE to pull from. Cloud Build let's you build the Docker container and put the image in Artifact Registry with a single command!

Cloud Build will compress the files from the directory and move them to a Cloud Storage bucket. The build process will then take all the files from the bucket and use the Dockerfile, which is present in the same directory, to run the Docker build process.

Create the target Docker repository
You must create a repository before you can push any images to it. Pushing an image can't trigger creation of a repository and the Cloud Build service account does not have permissions to create repositories.

In the console, search for Artifact Registry in the search field, then click on Artifact Registry result.

Click Create Repository.

Specify monolith-demo as the repository name.

Choose Docker as the format.

Under Location Type, select Region and then choose the location us-east1.

Click Create.

Configure authentication
Before you can push or pull images, configure Docker to use the Google Cloud CLI to authenticate requests to Artifact Registry.

To set up authentication to Docker repositories in the region us-east1, run the following command in Cloud Shell:
gcloud auth configure-docker us-east1-docker.pkg.dev
Copied!
The command updates your Docker configuration. You can now connect with Artifact Registry in your Google Cloud project to push and pull images.

Deploy the image
You will now deploy the image that was built earlier.

First you need to enable the Cloud Build, Artifact Registry, and Cloud Run APIs. Run the following command in Cloud Shell to enable them:
gcloud services enable artifactregistry.googleapis.com \
    cloudbuild.googleapis.com \
    run.googleapis.com
Copied!
After the APIs are enabled, run the following command to start the build process:
gcloud builds submit --tag us-east1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0
Copied!
Note: This process will take a few minutes.
To view your build history, or watch the process in real time, in the console, search for Cloud Build then click on the Cloud Build result.
On the History page you can see a list of all your builds; there should only be 1 that you just created.
Build History list

If you click on the Build ID, you can see all the details for that build including the log output.

From the Build Details page you can view the container image that was created by clicking the Execution Details tab, then clicking on on the image link.

Build details

Click Check my progress to verify the objective.
Assessment Completed!
Create Docker Container with Google Cloud Build
Assessment Completed!

Task 3. Deploy the container to Cloud Run
In this task, you deploy the containerized website from Artifact Registry to Cloud Run, a fully managed serverless platform that automatically handles infrastructure, scaling, and deployment.

Run the following command to deploy the image to Cloud Run:
gcloud run deploy monolith --image us-east1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0 --region us-east1
Copied!
When asked to allow unauthenticated invocations to [monolith] type Y.
Click Check my progress to verify the objective.
Assessment Completed!
Deploy Container To Cloud Run
Assessment Completed!

Verify deployment
To verify the deployment was created successfully, run the following command:
gcloud run services list
Copied!
Note: It may take a few moments for the pod status to be Running.
Output:

✔
SERVICE: monolith
REGION: us-east1
URL: https://monolith-2cxtmp4m2q-uc.a.run.app
LAST DEPLOYED BY: student-02-aa7a5aed362d@qwiklabs.net
LAST DEPLOYED AT: 2022-08-19T19:16:14.351981Z
This output shows several things. You can see the deployment, as well as the user that deployed it (your email) and the URL you can use to access the app. Looks like everything was created successfully!

Click on the URL provided in the list of services. You should see the same website you previewed locally.
Note: You can also view your Cloud Run deployments via the console if you navigate to Cloud Run in the Navigation menu.
Task 4. Create new revision with lower concurrency
In this section you will deploy your application again, but this time adjusting one of the parameters.

By default, a Cloud Run application will have a concurrency value of 80, meaning that each container instance will serve up to 80 requests at a time. This is a big departure from the Functions-as-a-Service model, where one instance handles one request at a time.

Run the following command to re-deploy the same container image with a concurrency value of 1 (just for testing), and see what happens:
gcloud run deploy monolith --image us-east1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0 --region us-east1 --concurrency 1
Copied!
To see the details, from the Navigation menu, click on Cloud Run, then click on the monolith service:
The monolith service

On the Service Details page, click on the Revisions tab. You should now see 2 revisions created.
The most recent deployment has Details on the right hand side.

The monolith Revisions tab

You will see that the concurrency value has been reduced to "1".

The monolith Container tab

Although this configuration is sufficient for testing, in most production scenarios you will have containers supporting multiple concurrent requests.

Click Check my progress to verify the objective.
Assessment Completed!
Create new revision with lower concurrency
Assessment Completed!

Next, you can restore the original concurrency without re-deploying. You could set the concurrency value back to the default of "80", or you could just set the value to "0", which will remove any concurrency restrictions and set it to the default max (which happens to be 80).

Run the following command to update the current revision, using a concurrency value of 80:
gcloud run deploy monolith --image us-east1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:1.0.0 --region us-east1 --concurrency 80
Copied!
You will notice that another revision has been created, that traffic has now been redirected, and that the concurrency is back up to 80.

Note: You may need to leave the Revisions tab and then return to it to see the most up to date information.
Task 5. Make changes to the website
Scenario: Your marketing team has asked you to change the homepage for your site. They think it should be more informative of who your company is and what you actually sell.

Task: You will add some text to the homepage to make the marketing team happy! It looks like one of our developers already created the changes with the file name index.js.new. You can just copy this file to index.js and your changes should be reflected. Follow the instructions below to make the appropriate changes.

Run the following commands to copy the updated file to the correct file name:
cd ~/monolith-to-microservices/react-app/src/pages/Home
mv index.js.new index.js
Copied!
Print its contents to verify the changes:
cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js
Copied!
The resulting code should look like this:

/*
Copyright 2019 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
import React from "react";
import { Box, Paper, Typography } from "@mui/material";

export default function Home() {
  return (
    <Box sx={{ flexGrow: 1 }}>
      <Paper
        elevation={3}
        sx={{
          width: "800px",
          margin: "0 auto",
          padding: (theme) => theme.spacing(3, 2),
        }}
      >
        <Typography variant="h5">Fancy Fashion &amp; Style Online</Typography>
        <br />
        <Typography variant="body1">
          Tired of mainstream fashion ideas, popular trends and societal norms?
          This line of lifestyle products will help you catch up with the Fancy
          trend and express your personal style. Start shopping Fancy items now!
        </Typography>
      </Paper>
    </Box>
  );
}
You updated the React components, but you need to build the React app to generate the static files.

Run the following command to build the React app and copy it into the monolith public directory:
cd ~/monolith-to-microservices/react-app
npm run build:monolith
Copied!
Now that the code is updated, rebuild the Docker container and publish it to Artifact Registry. You can use the same command as before, except this time you will update the version label.

Run the following command to trigger a new Cloud Build with an updated image version of 2.0.0:
cd ~/monolith-to-microservices/monolith
gcloud builds submit --tag us-east1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:2.0.0
Copied!
In the next section you will use this image to update your application with zero downtime.

Click Check my progress to verify the objective.
Assessment Completed!
Make Changes To The Website
Assessment Completed!

Task 6. Update website with zero downtime
The changes are complete and the marketing team is happy with your updates! It is time to update the website without interruption to the users. Cloud Run treats each deployment as a new Revision which will first be brought online, then have traffic redirected to it.

By default the latest revision will be assigned 100% of the inbound traffic for a service. It is possible to use "Routes" to allocate different percentages of traffic to different revisions within a service. Follow the instructions below to update your website.

Run the following command to re-deploy the service to update the image to a new version with the following command:
gcloud run deploy monolith --image us-east1-docker.pkg.dev/${GOOGLE_CLOUD_PROJECT}/monolith-demo/monolith:2.0.0 --region us-east1
Copied!
Click Check my progress to verify the objective.
Assessment Completed!
Update website with zero downtime
Assessment Completed!

Verify deployment
Validate that your deployment updated by running the following command:
gcloud run services describe monolith --platform managed --region us-east1
Copied!
Output:

✔ Service monolith in region 
Here you will see that the Service is now using the latest version of your image, deployed in a new revision.

To verify the changes, navigate to the external URL of the Cloud Run service, refresh the page, and notice that the application title has been updated.

Run the following command to list the services and view the service Url:
gcloud beta run services list
Copied!
Click on the URL of the service. Your web site should now be displaying the text you just added to the homepage component!
The updated Fancy Store website
