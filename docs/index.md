## Overview

This guide will walk you through the process of deploying a sample Java application in CFT.

An application will be generated from a template and deployed to a kubernetes cluster.

At the end of this tutorial, you will be able to access your application via VPN and will have made changes to it.

## Prerequisites

GitHub access is required to complete the steps in this tutorial. See the [onboarding guide](https://hmcts.github.io/onboarding/team/github.html#github) to get setup.


## Steps

#### Create a Repository and build pipeline

1. Click `Create` in the Backstage sidebar and select [`Spring Boot Service`](https://backstage.platform.hmcts.net/create) template. Prefix your GitHub repository name with `labs-*`. This enables Jenkins to automatically pickup the folder based on the GitHub repository name.
   
   Description and default values for various `Fields` in the template.
   
   - Product:  `labs`        #Product this component belongs to, normally the team name, e.g. cmc, labs
    
   - Component:            #Name of the component, e.g. backend
    
   - Slack contact channel:#Which channel (or user) to contact if there's any issues with this service.
    
   - Description:          #Description of the application, a sensible default will be used if not specified
    
   - HTTP port:            #The port to run the app on.
    
   - GitHub admin team:    #Which GitHub team should have admin permissions, use the format hmcts/<team-id>
   
   - Owner:                #Owner of the Component
   
   - Host:  `github.com`
   
   - Owner:                #The organization, user or project that this repo will belong to
   
   - Repository: `labs-<Component>`     #The name of the repository
   
    
2. Create Helm values file with following contents(this is only needed in the lab as the application will be run on sandbox).

   CFT: values.sandbox.template.yaml

    ```yaml
        java:
          # Don't modify below here
          image: ${IMAGE_NAME}
          ingressHost: ${SERVICE_FQDN}
    ```
   
3. Get the Pull Request reviewed and merged.

#### Build application

1. Login to Jenkins and select [HMCTS - Labs](https://sandbox-build.platform.hmcts.net/job/HMCTS_Sandbox_LABS/) folder.
Scan the organization by clicking on `Scan Organization Now`.
New repository should be listed under repositories after scan finishes.
Logs can be monitored under `Scan Organization Log`.
Any GitHub repository that starts with `labs*` will be listed as part of this scan.


2. Run the jenkins pipeline against the `master` branch.

#### Configure load balancing for HA

1. We load balance across AKS clusters using `Azure Application Gateway`. Add a couple of lines of config for the application in [config file](https://github.com/hmcts/azure-platform-terraform/blob/master/environments/sbox/backend_lb_config.yaml).

   ```yaml
   #labs
      - product: "labs"
        component:     # GitHub repository name without "labs" prefix, e.g. `mohanay`
   ```
     
#### Deploy application

1. We practise [GitOps](https://www.weave.works/technologies/gitops/) for application deployment to Kubernetes.
   Read the [documentation]( https://github.com/hmcts/cnp-flux-config/blob/master/docs/app-deployment-v2.md) and follow the instructions.

#### Access application

1. Access the deployed application using the URL.

   ```
   http://<product>-<component>-sbox.service.core-compute-sandbox.internal 
   ```  
   
#### Customise application

1. Helm Charts can be customised by updating corresponding values file for each environment. Update `values.yaml` file located under `/charts/<repo-name>` directory.  
 
2. Environment variables can be passed by updating the corresponding values file in Helm chart. 
 
   ```yaml
   java:
     environment:
       FAVOURITE_FRUIT: plum   # KEY must be in uppercase
   ```

## Troubleshooting

See our [troubleshooting](https://hmcts.github.io/ways-of-working/troubleshooting/#troubleshooting-issues) guide.

 - Pod/Application logs can viewed using `kubectl` command.
 - For Deployment issue check flux logs / resource status.  
        

## Slack Channels

- `#golden-path` is for support requests to the Platform Operations team
- `#labs-build-notices` jenkins build notices channel
