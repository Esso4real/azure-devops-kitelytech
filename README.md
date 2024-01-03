_DevOps Scenario_

_We have a client that is located in the United States and are in the financial industry because of this factor their code is required to be accessed only via state side server. Our developer group will be in India. The current legacy stack is Asp.Net 4.3 and will need to be upgraded to 4.8. All development will happen in Azure. Developers will need access to Visual Studio 2019, SMTP (outlook) SQLServer._

_Given this information please explain in detail how you would set up an environment for our offshore team to complete this work._


`In order to set up an environment for our offshore team located in India to complete this work, considering that our client is a financial industry, located in the U.S, where customers are conscious of platform-level security in addition to application level security. We'll have to deploy our applications using the Azure App Service Environment. To restrict application access from the internet, the Azure Application Gateway service and Azure Web Application Firewall are used. We'll also make use of a continuous integration and continuous deployment (CI/CD) for App Service Environments using Azure DevOps for a reliable and secure deployment. `

## Architecture

!image_URL_here

- Our developers working in India will connect to resources provisioned in the U.S region using an Internal LoadBalancer (ILB) from an  ExpressRoute / VPN for secure connection.

## Scenario

- We will build an Azure Web App where extra security is required.
- We provide dedicated tenancy, rather than shared tenant App Service Plans.
- We'll use Azure DevOps with an internally load-balanced(ILB) Application Service Environment.
Architecture


Below is a detail explanation on setting up the environment in Azure for a client base in the U.S region with developers in the Asia region (India), ensuring access only via U.S servers.

## Prerequisites:
### 1. Azure Subscription (with Contributor/Owner access) and Active Directory:
    - Ensure you have an Azure subscription that covers the U.S and Asia regions and Set up Azure Active Directory to manage user identities and access for developers.

## Infrastructure Setup:
For best practices, use Azure Resource Manager (ARM) templates to define the infrastructure as code or Terraform

### Deploy App Service Environment
    - Create a new Resource Group to hold all Azure resources related to this project.
    - Create virtual network (VNet) with address range, ex: 10.0.1.0/16 with subnet address range 10.0.1.0/24 to hold our CI/CD agent VM and  App Gateway with address range 10.0.2.0/24 for the subnet. (Note: We will want to leave a space in our VNet for the subnet holding the App Service Environment that we will create later, it is also best practice to allow space for a subnet for future use).

### Create App Service Environment (ASE)

    - Select the VNet you created earlier (do not let the ASE deployment create its own VNet as this gives you far less control over the network configuration)
    - Deploy the ASE into a new subnet with address range 10.0.0.0/24
    - Set the VNet IP Type to Internal to create an Internal Load Balancer (ILB) in front of the ASE
    - Set the Domain.
    - Note: when the ASE is provisioned it creates a Network Security Group (NSG) pre-configured with all the ports it needs to function fully. However, it does not attach the NSG to the subnet that the ASE is deployed to. You currently have to go back and manually attach the NSG to the subnet otherwise, when the application is deployed you will not be able to have the web app talk to the API app.
    - On the NSG that was created, open the Subnets and Associate it with the ASE's subnet.

### Create a certificate for the Internal Load Balancer (ILB)

## Deploy Web Apps
### Create Web Apps
    - Create a new ASE based Web App with the location set to the ASE.

### Create SQL Database
    - Create SQL Database and configure the SQL server with server name and admin credentials.
    - Turn Off the Allow access to Azure services setting (since we are accessing it only directly from within the VNet).
    - Enable the Service Endpoints or Private Links for your subnet.

### Create Storage Account:
    - Create Storage Account for storing development artifacts and backups.

## Deploy Agent VM
    - Create a Virtual Machine inside the CI/CD subnet that we can use for testing. This VM will later be used to run a VSTS Agent that will build and release our application to our App Service Environment. Since the ASE is not publicly exposed to the internet and therefore unreachable from VSTS, we will need an agent running inside the VNet that is hosting the application.

    - In the marketplace search and create a VM with Pre-installed Visual Studio 2019.
    - Next we need to create a Network Security Group to allow RDP / SSH access depending on the VM type.

### Alternatively
    - Create an Azure Virtual Network (VNet) in the US region and divide the VNet into appropriate subnets, such as one for US servers and another for Asia servers. Note: For best practices is always a good practice to create SubNets with more address space in case we need additional resources in the future. 
    - Use NSGs (firewall rules) to control traffic between subnets and enforce access rules. Allow access only from the US subnet to the Asia subnet, restricting access the other way, or allow access only from designated IP ranges, enhancing security, or consider using a VPN for secure communication between the US and Asia regions (optional).
    - Set up Azure Bastion in the US subnet for secure remote access to servers.
    - Provision Azure VMs in the US region and ensure VMs meet the hardware and software requirements for Asp.Net 4.8 app.
    - Choose appropriate VM sizes based on the client's requirements for performance.

### SMTP Configuration
    - Set up an SMTP relay server and Configure necessary credentials and secure connections within Azure for handling Outlook-based email communication.

### Access Management:
    - Integrate Azure AD with on-premises AD (if applicable) for user management.
    - Define RBAC roles and assign appropriate permissions to developers based on their responsibilities.
    - Use Azure Key Vault to securely manage and store sensitive information such as database connection strings and API keys.

## Deploy App Gateway
    - Create App Gateway and configure it to connect with internal IP Address of the ASE's ILB  
    - Integrate your ILB App Service Environment with the Azure Application Gateway
    - The App Service Environment is a deployment of Azure App Service in the subnet of a customer's Azure virtual network. It can be deployed with an external or internal endpoint for app access. The deployment of the App Service environment with an internal endpoint is called an internal load balancer (ILB) App Service environment (ASE).
    - Web application firewalls help secure your web applications by inspecting inbound web traffic to block SQL injections, Cross-Site Scripting, malware uploads & application DDoS and other attacks. You can get a WAF device from the Azure Marketplace or you can use the Azure Application Gateway.
    - The Azure Application Gateway is a virtual appliance that provides layer 7 load balancing, TLS/SSL offloading, and web application firewall (WAF) protection. It can listen on a public IP address and route traffic to your application endpoint. The following information describes how to integrate a WAF-configured application gateway with an app in an ILB App Service environment.
    - The integration of the application gateway with the ILB App Service environment is at an app level. When you configure the application gateway with your ILB App Service environment, you're doing it for specific apps in your ILB App Service environment. This technique enables hosting secure multitenant applications in a single ILB App Service environment.


### Configure DNS to access the webapp using the gateway

## Setup CI/CD

### Application Migration and Upgrade / DevOps Setup:
    - Upgrade the legacy ASP.NET application from version 4.3 to 4.8. This might involve code changes, library / dependency updates, and testing.
    - Utilize Azure Boards for work tracking, ensuring transparency and efficient project management.
    - Create an Azure DevOps project.
    - Use Azure DevOps for version control, continuous integration, and deployment. Create pipelines for automated builds and releases.
    - Store the upgraded code in a secure source code repository within Azure DevOps or use any other private SCM tool like GitHub, GitLab, BitBucket.
    - Automate compilation and packaging of Asp.Net 4.8 application by building pipeline in Azure DevOps and Integrate automated testing for code quality and functionality.
    - Establish a release pipeline with stages for Dev, Test, Staging and Production.
    - Integrate approvals in critical stages.

### Monitoring and Logging:
    - Set up Azure Monitor to track VM performance, application logs, and security events.
    - Configure Log Analytics to centralize logs for analysis and troubleshooting.

### Compliance:
    - Ensure compliance with relevant industry standards and regulations.
    - Implement encryption for data in transit and at rest to meet security requirements.

### Testing and Validation:
    - Conduct UAT with the Asia (India) development team to ensure the environment meets project requirements.

### Documentation and Knowledge Transfer:
    - Create comprehensive documentation for the environment setup, configurations, and troubleshooting steps and provide training sessions for the Asia development team on the new environment, tools, and processes.

By following these steps, you can set up a secure and segregated environment in Azure, allowing access only via US servers while meeting the development and communication needs of the client and developers in the US and Asia regions, respectively. Regular monitoring, maintenance, and adherence to best practices will contribute to the overall success of the project.
