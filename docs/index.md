## Overview
This guide walk-through the process of deploying an Sample Java application in CFT environment.

This guide uses a pre-configured application from a template. Application is deployed to a kubernetes cluster. 
At the end of this tutorial, you should be able to access an working application via VPN and made changes to it.

## Prerequisite

`Github` access is required to complete the steps in this tutorial. Following the [link](https://hmcts.github.io/onboarding/team/github.html#github) to request access.

## High Level Steps

- Create a Repository and build pipeline
- Build application
- Configure load balancing for HA
- Deploy application
- Access application
- Customize application
- Troubleshooting

## Steps

#### Create a Repository and build pipeline

1. Access [Backstage](https://backstage.platform.hmcts.net/create) to create a new github repo by selecting predefined [`Spring Boot Service`](https://backstage.platform.hmcts.net/create/templates/springboot-template) template. Please make sure name of  Github repo  starts with `labs*`. This enables Jenkins build tool to organize the folder based on Github repo names.

2. Create Helm values file with following contents(this is only needed in the lab as the application will be run on sandbox).

   CFT: values.sandbox.template.yaml
   
    ```
        java:
          # Don't modify below here
          image: ${IMAGE_NAME}
          ingressHost: ${SERVICE_FQDN}
   
3. Get the Pull Request reviewed and merged.

#### Build application

1. Login to Jenkins and select "HMCTS - Labs" folder. Scan the organization by clicking on `Scan Organization Now`. New repository should be listed under repositories after scan finishes. Logs can be monitored under `Scan Organization Log`. Any GitHub repository that starts with `labs*` will be listed as part of this scan.

   CFT: [jenkins](https://sandbox-build.platform.hmcts.net/job/HMCTS_LABS/)


2. Run the jenkins pipeline against the `master` branch.

#### Configure load balancing for HA

1. We load balance across AKS clusters using `Azure Application Gateway`. Add a couple of lines of config for the application.

   CFT:  [config file](https://github.com/hmcts/azure-platform-terraform/blob/master/environments/sbox/backend_lb_config.yaml)

      ```
           #labs
               - product: "labs"
                 component:     #githubreponame without "labs" prefix
   
#### Deploy application

1. We practise [GitOps](https://www.weave.works/technologies/gitops/) for application deployment to Kubernetes.

   CFT: Refer to [documentation.]( https://github.com/hmcts/cnp-flux-config/blob/master/docs/app-deployment-v2.md)

#### Access application

1. Access the deployed application using the URL.

    ```
      http://<product>-<component>-sbox.service.core-compute-sandbox.internal   
   
#### Customize application

1. Helm Charts can be customised by updating corresponding values file for each environment. Values files located under `/charts/<repo-name>/values.<ENV>.template.yaml`  
 
2. Environment variables can be passed by updating the corresponding values file in Helm chart. 
 
       
       java:
         environment:
           FAVOURITE_FRUIT: plum   # KEY must be in uppercase

#### Troubleshooting

1. Troubleshooting

     [Follow](https://hmcts.github.io/ways-of-working/troubleshooting/#troubleshooting-issues) the link to refer to troubleshoot steps.
     - Pod/Application logs can viewed using `kubectl` command.
     - For Deployment issue check Flux.  
        
        

## Slack Channels

- `#golden-path` is for support requests to the Platform Operations team
- `#labs-build-notices` jenkins build notices channel


