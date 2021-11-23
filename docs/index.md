## Overview
This guide walk-through the process of deploying an Sample Java application in Cloud Platform.

Before working through the tutorial, we recommend reading following pages and assume you have all the required accounts.

- https://hmcts.github.io/onboarding
- https://hmcts.github.io/ways-of-working/

This guide uses a pre-configured application from an template which can be deployed from [Backstage](https://backstage.platform.hmcts.net/). Application is deployed to AKS(kubernetes) cluster inside `labs` namespace in sbox/sandbox environment.

## Steps

1. Access [Backstage](https://backstage.platform.hmcts.net/create) to create a new github repo containing application template. Please make sure name of  Github repo  starts with `labs*`. This enables Jenkins build tool to organize the folder based on Github repo names.

2. Create `Jenkinsfile_parameterized`  file in the repo with following contents.
    ```
        #!groovy
        
        @Library("Infrastructure")
        
        def type = "java"
        def product = "labs"
        def component = "mohanalatest"
        
        withPipeline(type, product, component) {
        }
3. Create Helm values file with following contents.

   CFT: values.sandbox.template.yaml

   SDS: values.sbox.template.yaml
   
    ```
        java:
          # Don't modify below here
          image: ${IMAGE_NAME}
          ingressHost: ${SERVICE_FQDN}
4. Get the Pull Request reviewed and merged.

5. Login to Jenkins and select "HMCTS-Labs" folder. Scan the organization by clicking on `Scan Organization Now`. New repository should be listed under repositires after scan finishes. Logs can be moinitored under `Scan Organization Log`. Any github repos that starts with `labs*` will be listed as part of this scan.

   CFT: [jenkins](https://sandbox-build.platform.hmcts.net)

   SDS: [jenkins](https://sds-sandbox-build.platform.hmcts.net)

6. Run the jenkins pipeline against the `master` branch.

7. We load balance across AKS clusters using `Azure Application Gateway`. Add couple of lines of config for the application.

   CFT:  [config file](https://github.com/hmcts/azure-platform-terraform/blob/master/environments/sbox/backend_lb_config.yaml)

   SDS:  [config file](https://github.com/hmcts/sharedservices-azure-platform/blob/master/environments/sbox/backend_lb_config.yaml)
    ```
           #labs
               - product: "labs"
                 component:     #githubreponame without "labs" prefix

8. Azure Front Door is our entry point in HMCTS. Add following line of code for the app.

   CFT: [config file](https://github.com/hmcts/azure-platform-terraform/blob/master/environments/sbox/sbox.tfvars)
    ```
     {
        product          = "labs"
        name             = "#githubreponame without "labs" prefix"
        custom_domain    = "your-app.service.core-compute-sandbox.internal"
        backend_domain   = ["firewall-sbox-int-palo-sbox.uksouth.cloudapp.azure.com"]
        certificate_name = "wildcard-sandbox-platform-hmcts-net"
      }
    ```
   
   SDS: [config file](https://github.com/hmcts/sharedservices-azure-platform/blob/master/environments/sbox/sbox.tfvars)

    ```
       {
        product          = "labs"
        name             = "#githubreponame without "labs" prefix"
        custom_domain    = "your-app.sandbox.platform.hmcts.net"
        backend_domain   = ["firewall-sbox-int-palo-sdsapimgmt.uksouth.cloudapp.azure.com"]
        certificate_name = "wildcard-sandbox-platform-hmcts-net"
      }


9. We practise [GitOps](https://www.weave.works/technologies/gitops/) for application deployment to Kubernetes.

   CFT: [repo](https://github.com/hmcts/cnp-flux-config)

   SDS: [repo](https://github.com/hmcts/shared-services-flux)

10. Access the deployed application using the URL mentioned in FrontDoor code.
    ```
    custom_domain    = "your-app.sandbox.platform.hmcts.net"   
   
11. Helm Charts can be customized by updating  corresponding values file for each environment. Values files located under `/charts/<repo-name>/values.<ENV>.template.yaml`  
 
12. Environment variables can be passed by updating the corresponding values file in Helm chart. 
 
       ```
       java:
         environment:
           FAVOURITE_FRUIT: plum   # KEY must be in uppercase


13. Troubleshooting

     [Follow](https://hmcts.github.io/ways-of-working/troubleshooting/#troubleshooting-issues) the link to refer to troubleshoot steps.
     - Pod/Application logs can viewed using `kubectl` command.
     - For Deployment issue check Flux.  
        
        

## Slack Channels

- `#platops-labs` is for support requests to the Platform Operations team
- `#labs-build-notices` jenkins build notices channel


