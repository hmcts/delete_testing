## Overview

This guide will walk you through the process of deploying a sample Java application in CFT.

An application will be generated from a template and deployed to a kubernetes cluster.

At the end of this tutorial, you will be able to access your application via VPN and will have made changes to it.

## Prerequisites

GitHub access is required to complete the steps in this tutorial. See the [onboarding guide](https://hmcts.github.io/onboarding/team/github.html#github) to get setup.


## Steps

#### Create a Repository and build pipeline

1. Click `Create` in the Backstage sidebar and select [`Spring Boot Service`](https://backstage.platform.hmcts.net/create) template. 

   Default values for various `Fields` in the template.
   
   
   - Page-1
   
       - Product:                       `labs`
           
       - Component:                     `YourGithubUsername`
        
       - Slack contact channel:         `cloud-native`
            
       - Description:                   `Deploying a Java application`
        
       - HTTP port:                     `80`
        
       - GitHub admin team:             `hmcts/reform`
       
       - Owner:                         `dts_cft_developers`
       
       
   - Page-2
   
       - Host:                          `github.com`
       
       - Owner:                         `hmcts`
       
       - Repository:                    `labs-YourGithubUsername`
 

#### Build application

1. Login to Jenkins and select [HMCTS - Labs](https://sandbox-build.platform.hmcts.net/job/HMCTS_Sandbox_LABS/) folder.
Scan the organization by clicking on `Scan Organization Now`.
The new repository should be listed under repositories after the scan finishes.
Logs can be monitored under `Scan Organization Log`.
Any GitHub repository that starts with `labs-*` will be listed as part of this scan.


2. Run the jenkins pipeline against the `master` branch.

#### Configure load balancing for HA

1. We load balance across AKS clusters using [Azure Application Gateway](https://docs.microsoft.com/en-us/azure/application-gateway/overview). Add a couple of lines of config for the application in [config file](https://github.com/hmcts/azure-platform-terraform/blob/master/environments/sbox/backend_lb_config.yaml).

   ```yaml
   #labs
      - product: "labs"
        component:     # GitHub repository name without "labs" prefix, e.g. `YourGithubUsername`
   ```
     
#### Deploy application

1. We practise [GitOps](https://www.weave.works/technologies/gitops/) for application deployment to Kubernetes.

   Application will be deployed in `labs` kubernetes namespace which has been already been created. 
   Follow the app deployment [guide](hmcts/cnp-flux-config@master/docs/app-deployment-v2.md#add-a-new-application) in cnp-flux-config.

#### Access application

1. Access the deployed application using the URL.

   ```
   http://<product>-<component>-sbox.service.core-compute-sandbox.internal 
   ```  
   
#### Customise application

We are going to customise the application by changing default landing page for the application by passing environment variables. 

1. Helm Charts can be customised by updating `values.yaml` file located under `/charts/<repo-name>` directory.  

2. Environment variables can be passed by updating values file in the Helm chart. 
 
   ```yaml
   java:
     environment:
       FAVOURITE_FRUIT: plum   # KEY must be in uppercase
   ```
3. Update code to reference environment variable in `RootController.java` file located under `src/main/java/uk/gov/hmcts/reform/<Component>/controllers/`.

   ```java
    public ResponseEntity<String> welcome() {
        return ok("Welcome to your app, my favourite fruit is " +  System.getenv("FAVOURITE_FRUIT"));
    }
   ```
4. Ask someone on your team to review your `pull request` and then merge it.

5. Run the Jenkins pipeline against the `master` branch (this will trigger automatically on the production Jenkins instance).

6. Access the application using same `URL`.


## Troubleshooting

See our [troubleshooting](https://hmcts.github.io/ways-of-working/troubleshooting/#troubleshooting-issues) guide.
        

## Slack Channels

- `#golden-path` is for community discussion about the tutorials.
- `#labs-build-notices` jenkins build notices channel.
- `#platops-help`   is for raising support requests to the `Platform Operations` team.
