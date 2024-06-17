---
slug: 5-ways-platform-eng
title: 5 ways to dramatically speed up your cloud application teams
tags: [platform engineering, azure, bicep, identity]
image: preview.png
authors: [keith]
---

Working with applications teams and partners developing cloud native apps on Azure, you quickly learn developer time is valuable, enthusiasm & flow state is critically important. 

Whenever an application team has to _wait_ for an environment, _wait_ for a access, a service now ticket, a support case, admin access to install tooling, productivity is dramatically effected, projects can be 2x longer and of lower quality

Equally, it's important to have a designed, governed environment when using the public cloud, so your workload teams start right, and stay right! This covers all the normal design pillars of a well architected solution, reliability, security, cost and performance.   

Application teams work best when they can select their preferred platform services, tooling, languages and libraries, and most importantly, reduce their dependencies & constraints. To this end, platform teams number one priority should be to work towards a self-service model, removing themselves from the process, constantly unblock friction points, while meeting the application teams where they are, rather than imposing platform and tooling assumptions.  


## Where Platform teams should focus


You don't need to have everything automated from day 1, nor have tooling for everything, but focusing in these 5 crucial elements will result in more impact for everyone in your organization, while avoiding unnecessary tickets/cases and bottlenecks, something I have seen depressingly way to often.


### 1. Environment Provisioning 

When a application team works on a new product, timely access to an environment is important when enthusiasm is high.  You should be targeting giving teams access to an environment they can use within 30mins from the initial request.

This would look like vending [Resource Groups](https://learn.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal#what-is-a-resource-group) within shared Subscriptions, provisioning new Subscription, when needed with a good tagging model.


:::tip
Subscriptions now in Azure can support hundreds of developers, the subscriptions have granular role-based controls, mature cost tracking services, and you can now track [Subscription Limits](https://learn.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#subscription-limits) & usage very effectively.  
:::

This can avoid Subscription sprawl, so you end up with a list that is meaningful and manageable. There are a number of good resources on [naming recommendations](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)

During the vending process, it will be important to capture:
 * Workload Type (e.g. __Production vs Sandbox__),  this drives the policies that are applied to control what can be deployed & levels of access, and keeps the separation between these environments. Keep the separation of any Production and Sandbox environments should be at the subscription level.
 * Required Networking (e.g. __Connected or Non-connected__) this determines if private IP connectivity is required by the workload, or if ingress/egress to the workload needs to be privately routed.

The first, simplest, most unconstrained environment you should offer is a  Non-connected Sandbox, this allows the application teams the most flexibility to experiment with multiple services, full access to the environment in the portal to allow the team to rapidly get ideas to a POC stage. Here typically, there are no or little restrictions on access or resources that can be provisioned. The most constrained, and complex environment will be a Connected Production subscription, this will have policies to ensure production guardrails are followed, and networking to allow private IP connectivity, and ingress/egress routing controls (if needed).

:::tip
The new [Subscription Vending Bicep Verified Module](https://github.com/Azure/bicep-registry-modules/tree/main/avm/ptn/lz/sub-vending) is a excellent starting point to start vending these environments, from the simplest to the most complex with a module & parameter driven approach.  You can collect the required information from the application team, then call the vending module directly from the `az cli` to start with, or create a pipeline/action in your favorite devops tool, maybe trigger a GitHub workflow from a Issue template:

![alt text](example-issue.png)
:::


:::tip
__Hot Take #1__:   I'd recommend Bicep over Terraform every time when automating environment provisioning or application deployment on Azure, even if you are multi-cloud! It's a simple, powerful, performant 1st class experience,  without needing the complexities of a state file, as the state is whatever is deployed in azure, and templates can be re-run and only the changes will be deployed. 
:::



### 2. Environment Permissions

So you have vended an environment, the app team tried to provision their first internally authenticated webapp that calls a OpenAI gpt-4o model using identity based access, deployed using github actions...  <span style={{color: 'red'}}>error error error</span> 4 tickets in 5 minutes, now the team are googling for workarounds, not delivering their projects, wasting valuable time, and enthusiasm.  Whats the problem?

	* No permissions to create a [Role Assignment](https://learn.microsoft.com/azure/role-based-access-control/role-assignments) on the webapp managed identity
	* OpenAI resource not [registered](https://learn.microsoft.com/azure/azure-resource-manager/troubleshooting/error-register-resource-provider?tabs=azure-portal) in subscription
	* Cannot create [Application Registration](https://learn.microsoft.com/entra/identity/role-based-access-control/delegate-app-roles#restrict-who-can-create-applications) in EntraID
	* Require [Admin consent](https://learn.microsoft.com/entra/identity/enterprise-apps/manage-consent-requests#evaluate-a-request-for-tenant-wide-admin-consent) for application permissions.
	* Cannot create [federated access from github](https://learn.microsoft.com/azure/developer/github/connect-from-azure) to deploy to Azure

When building cloud native apps, __managed identity and role based access__ is a crucial part of the application architecture, and 100% the best and most secure way of creating cloud native applications.

Platform teams __must__ provide the appropriate level of access to the application team to allow these solution architectures.  I've seen this being the single thing that wastes tens/hundreds of hours of skilled peoples time

#### Recommendation #1
When assigning roles to the application team, ```Contributor``` is not enough to create identity-based solution architectures! Look to provide the team ```Contributor``` &  ```Role Based Access Control Administrator```, this role can be scoped to either the resource group or the subscription (depending on what you are vending), and can be further [limited](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-portal#step-5-(optional)-add-condition) to only assign selected roles to selected principals.

#### Recommendation #2
If vending Resource Groups, ensure resource provider [registrations](https://learn.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider) have been done as part of the vending process, and not blocking the application teams from creating their resources.

#### Recommendation #3
Many applications will need users to authenticate, and the best way of doing that is with Entra ID.  These apps need [application registrations](https://learn.microsoft.com/entra/identity-platform/quickstart-register-app) within EntraID, if your organization blocks the self-service creation of new application registrations, and/or has restrictive consent granting.  Ensure the team know the process for requesting a new application registration. Also, unless you want a new Service now ticket every time the app team what to add a new callback uri, make then a owner of the app registration in the process.

#### Recommendation #4
Lastly, yes, ensure the use of [Identity & Automation](https://learn.microsoft.com/azure/developer/github/connect-from-azure) for deployments in Production environments, but don’t take away access to the portal from your application teams! Grant your application team corporate identities roles in the environment.  Not granting this access, especially in the lower environments will make it much harder for the teams.

:::note
This article doesn't cover Namespace vending on a shared AKS cluster, I've seen a number of companies do this very well when there is a Kubernetes-first/only approach to the public cloud, in highly regulated industries, but it does need more engineering beyond the scope of this article, using AKS as a building block.  scale-to-zero container services like azure container apps is a great alternative that can be simply deployed into single team environment
:::

### 3. A little less Documentation & a little more sample repos

You will notice that our environment provisioning doesn’t make any assumptions about the application teams solution architecture, don’t couple the environment provisioning with the application provisioning, don’t make the application team deploy a AKS cluster to stand up a Static webapp.  This is not optimal use of the public cloud, it will not optimize your public cloud costs, nor minimize the required operation to support a product/app. Equally, don’t assume the structure or number of repo's needed.

Rather than documents, start to foster a innersource repo of samples, that can be simply provisioned into the vended environment, to show what a static webapp, or a simple microservices app, or an event driven process, or a integration workflow could look like.

These examples, with good READMEs can also inform the teams how to structure there application team repos.  

Look at the [Azure Developer CLI templates](https://learn.microsoft.com/azure/developer/azure-developer-cli/azd-templates?tabs=csharp) is a good example of this, I'm not saying you should use this tool, but `azd template list` shows this can look like (see screenprint),  there are way too many here, but you can start to create a cut down curated list that demonstrates getting started repos in each of the application solution categories.

![alt text](image.png)

Another thing to notice/adopt, in these samples repo ```/infra``` folders, their bicep is just composing a number of modules, these modules represent the 'right' way of configuring each service for your organization, for example, pre-configured with private endpoints and rbac base access.  To build a repo of these modules approved for use in your organization is recommended, you can also get started by using [Azure Verified Modules](https://azure.github.io/Azure-Verified-Modules/)


### 4. Tooling / Local loop development

Application teams experiment in the portal, develop locally, and provision from their local machine, then, add the automation and the managed identity to perform auto deployments via source control to apply in the later environments.

Ensure the teams can install/configure  VS code /  VS code extensions / command line tools / docker locally, and they have connectivity from these tools to the public cloud APIs they need. 


[```@azure/identity```](https://learn.microsoft.com/en-gb/azure/aks/workload-identity-overview#azure-identity-client-libraries) libraries are now brilliant! Most of the time, there is no need any more to use API keys or credentials that need to be rotated, we now just use Identities & RBAC.  Using these identity libraries, If the developer wants to run their code locally, and connect to a database in azure, the locally running app will operate with the local developers corporate identity (as per 'az login'), and as long as the dev has the appropriate RBAC on the database, all good.  If they deploy their app to Azure PaaS Service, without any code changes, the code will access the database using the services managed identity.  This makes the apps secure and resilient, and can be prompted up to production securely.
  

Without these tools and access, the application teams will not be writing the most secure way of coding their app. Another example where restricting application teams access can lower the security of their resulting workflows & applications.


### 5. Track Metrics

Track anything that causes friction.  Anytime the application team is waiting on something, a case, access to a service, resolving a bug, track it, dashboard it, and constantly priorities securely removing friction.  Promote the creation of issues on the platform teams repo, keep a prioritized backlog. Hold monthly feedback sessions.

Application teams will try to work around issues to ship their product, this can mean using the wrong environment, or using a less than ideal service or configuration, this hides inherent issues with the platform.


## Wrapup

Let me know what you think of these recommendations, if you are in a application team deploying to Azure, have a chat with the team providing you the environment, put in a monthly sync.  Its important the teams collaborate to get your companies products out the door, security, reliably and on time. 

