# Capstone Project Option 3 B-Safe - Project Write Up

## Project Description

Create a CI/CD Pipeline to convert the legacy development process to a DevOps process.
Background of the problem statement:
A leading US healthcare company, Aetna, with a large IT structure had a 12-week release cycle and their business was impacted due to the legacy process. To gain
true business value through faster feature releases, better service quality, and cost optimization, they wanted to adopt agility in their build and release process.
The objective is to implement iterative deployments, continuous innovation, and automated testing through the assistance of the strategy.

## Project Goal

Develop an automated pipeline for an application to build, test, and then deploy  

### Environment

* Jenkins instance setup in EC2
* Jenkins baseline instance
* Plugins
  * Groovy and Shared Libraries
  * Docker
  * Git
* Private Docker repo
* Execution environment
  * Docker installed
  * JDK
  * Maven
* Source SCM
  * Existing legacy project [RetailOne]

#### Environment Setup

Given that past projects have used Terraform to setup the EC2 instances, install them, and control networking permissions, that seemed the logical route to go for this project.
A Virtual Private Cloud (VPC) was created, consisting of:

* A gateway to allow external IPs and access
* A Build host configured with
* A Jenkins instance configured with the necessary plugins
* Internal subnet
* AWS key pair exchange
* An execution host for the deployed container

### Pipeline

A Jenkins server will perform the following:

* Be executed on demand, via both manual execution and polling of the source control manager
* Dynamically create a build/test container
  * Configured build environment
  * Application source and test code
  * Port access to monitor and log results
* Push image to private docker repository
* Build the application within the container
* Capture build results
* Execute the Unit test associated with the application
* Capture Unit test results
* Remove build/test container

## Project Setup

As a proof of concept, a pre-existing docker image was picked to serve as the base image, that already has front end and test cases that could be executed.
They are in the GitHub repo under the docker folder. 

## Docker Setup

A dockerfile was created to host the war inside the container, pulling in the pre-existing docker image mentioned before. It runs inside Tomcat and exposes the needed ports. 

## Pipeline Setup

The main guts of the project is handled via a Jenkins pipeline defined using a Groovy script.  The contents of the pipeline are stored in a xml file at:
https://github.com/stephenwatson888/Capstone/blob/main/JenkinsFile/config.xml

There are several stages within the pipeline, and in order they are:

* Get the code
* Build the code
* Run the tests
* Package the Webapp
* Build the image
* Verify local registry
* Push to local registry
* Deploy container


There is a second file called config2.xml, that calls out to a Jenkinsfile instead of embedding the script directly into the pipeline itself. This is merely a different way of creating the same pipeline, but both files are stored in the repo. 

## Infrastructure as Code Environment Setup

Again, I've grown fond of using Terraform from previous projects, leveraging Infrastructure as Code to automate as many pieces as possible. So even though Terraform wasn't explicitly called out as a requirement for this project, since I already had a working file that would stand up a Jenkins instance, I used that as the baseline for this project and merely added installation steps to add the pipeline to the server. These files can be found in the Jenkins folder in the GitHub repo. 

The biggest changes from previous projects are the three script files, installJenkins.sh, installPlugins.sh, and installWar.sh. These scripts are copied to the EC2 instance while Terraform is running and allow you to execute arbitrary commands in bash on the system. This gives you reasonably fine grained control over how the server is configured after being created so you can nail down the details of the server.

## Finalize the Jenkins Instance

Unfortunately, despite being able to execute arbitrary commands in BASH, Jenkins still requires you to log in to the web interface at least once to finalize the installation process. However once you do that you can go verify that the pipeline was installed correctly and is functional. 

### Verify Pipeline

Go to the Jenkins Dashboard and select the pipeline.

You will have to manually update the Jenkins Pipeline prior to running due to the dynamically allocated IP addresses within AWS. These changes can be made by hand from within the web browser easily enough. Then blick the "Build now" button and the pipeline should fire off and run successfully. 

## Verify the Application

Once the pipeline completes you should be able to verify the application by navigating to it in a web browser. Simply go to the page at:
http://0.0.0.0/retailone/
but replacing the 0.0.0.0 with the allocated IP address. 

## Forward work

I think the biggest step forwards for this project would be to use some sort of Jenkins image rather than a fresh install. I personally hate that you have to open Jenkins in a web browser the first time and I'd really like to automate that step away so that my Terraform script can truly execute everything. So I need to either use a base Jenkins image that's already past that step, or dig through the Jenkins files and see what changes after you load it for the first time and make those changes myself. 

The second step would be to programatically save the IP addresses assigned by the AWS Elastic IP, and modify the Jenkins Pipeline files to use those IP's instead of having to do it manually. Again, this is to drive my goal that I want to just run "terraform apply" and have an entirely working build at the end, with no other human intervention. I think this is possible but would require additional work and fiddling to make it functional. 
