## How to provision a Hybrid App Service Environment behind a WAF-Enabled Application Gateway




# Introduction
You may have heard of the Azure Application Gateway which is a Layer-7 HTTP/HTTPS load balancer that provides application-level routing and load balancing services that let you build a scalable and highly-available web front end in Azure. It works by accepting traffic and based on rules that are defined with it, routes the traffic to the appropriate back-end endpoints such as web apps and API's

•	Web Application Firewall (WAF)
•	HTTP /HTTPS load balancing
•	Cookie-based session affinity
•	SSL offloading and End-to-end SSL
•	URL-based content routing
•	Multi-site routing
•	WebSocket support
•	Health monitoring
•	TLS settings (Cipher Suites)
•	OWASP Profile

# Apps Service Environment Overview
Azure App service environments are essentially an azure app service feature that provides fully isolated and dedicated environments for running app service apps which are highly scalable. There are additional benefits to running your apps in an isolated environment such as managing traffic flow into the platform, managing compliance standards. 
In this situation, we are going to utilise the internal load balancer on the ASE for hybrid usage back to on-premise.
More information can be found here regarding azure app service environments. https://docs.microsoft.com/en-us/azure/app-service/environment/intro



# Architecture Objective
The end goal is to provide an App Service Environment with an internal load balancer which will be able to route traffic back to the on-premise network via VNet / express route routing

![Architecture](https://i.imgur.com/oRdpMBI.png)

Provision Application Service Environment
1.	Create a Resource Group
2.	Create VNet with applicable subnets, configure VNet DNS properties to lookup DNS locally.
Hint: Try to create a subnet dedicated to ASE with the maximum amount of address’s available in order to enable hyperscaling.

![sub](https://i.imgur.com/OXKLLvm.png)

3.	Provision Application Service Environment with Internal Domain Name (contoso.local). Remember to select “VIP” type to internal. As an internal load balancer is required for internal routing across VNet’s

![domain](https://i.imgur.com/wFZeVfd.png)

4.	Once your configuration has passed validation, sit back and enjoy some coffee as the deployment process for an ASE can take up to 4 hours.

5.	Once provisioned create Self Cert for ILB (https://docs.microsoft.com/en-gb/azure/app-service/environment/create-ilb-ase#post-ilb-ase-creation-validation)

6.	Once you have exported the certificate upload the certificate in the settings section within the ASE under ILB certificate.
7.	Navigate to “IP Address” within the ASE settings blade, Capture the “Internal Load Balancer IP Address)
8.	Create an Internal DNS record for the ASE domain name to point “*.asexx.contos.local”
9.	Import internal self-cert certificate to trusted root store for end clients web browsers
10.	Create Azure DevOps deployment agent (privately hosted or public cloud hosted) within the ci/cd subnet and configure appropriately via the Azure Dev Ops site (VSTS).
11.	 In the App Service Environment, create an App Service Plan and create a new Web App with the hostname of “yoursite.internal.contso.local” 
12.	 Test web app service via your web browser local. (This is under the assumption that you can route traffic from your local client to ASE VNet)

## Provision Azure Application Gateway (WAF-Enabled)
# Introduction: Azure Application Gateway (WAF-Tier)

Application gateways are great to route web traffic to your back end azure resources. With a combination of WAF capabilities & web traffic routing, managing your web traffic has become essential with public cloud apps. For more information on Azure application gateway’s head over to https://docs.microsoft.com/en-us/azure/application-gateway/overview
In this instance, we are going to provision an application gateway to allow public access to our backend endpoints in our App Service Environment (Isolated)  via the ASE internal load balancer.

1.	Provision Application with the following settings

![waf](https://i.imgur.com/YCS0ApK.png)

2.	Create a CName record (Externally) to point to application gateway public endpoint.  (DNS Lable) 
3.	Configure backend pool rule to point to the IP address of the ILB on the ASE
4.	Create NSG rule on the VNet /Subnet which the APFW is situated with the following information to allow the health probe to communicate with the backend app. (https://docs.microsoft.com/en-us/azure/application-gateway/application-gateway-faq#are-network-security-groups-supported-on-the-application-gateway-subnet)
5.	Create a Multi-Site Listener on the Application Gateway:
6.	Name - <webapp_name>HttpsListener
7.	Host Name - External Hostname in Change Description. 
8.	Protocol - HTTPS
9.	Certificate – Upload public certificate for external encryption 

10.	Create a Health Probe on the Application Gateway
a.	Name - <webapp_name>Probe
b.	Protocol - HTTPS
c.	Host - Backend Web Application Hostname 
d.	Path – “/”
11.	7.) Create an HTTP Setting on the Application Gateway
a.	Name - <webapp_name>HttpSetting
b.	Protocol - HTTPS
c.	Backend Auth Certificate – Use or upload backend certificate from step 5 of the ASE provisioning steps.
d.	Use Customer Probe - Select Probe from Step 5

12.	8.) Create a Basic Rule on the Application Gateway
a.	Name - <webapp_name>Rule
b.	Listener - From Step 2
c.	Backend Pool - ilb-ase-pool
d.	Setting - From Step 6

13.	9.) Configure App “ASE” Custom Domain Settings to match the external hostname
14.	10.) Test external hopefully you should get the below the result

![webapp](https://i.imgur.com/liOKciW.png)

15.	Check the backend health via the monitoring pane to confirm pass through of traffic routing. 


 

## Summary:

After the implementation, provisioning process of the App service environment & application, WAF enabled gateway you should be able to access your web app externally. Your application is now encrypted with TLS standards and presented via HTTPS Load Balancer.  You can configure your SSL policy via the listener's blade within the app gateway against your information security standards. 

Don’t forget to enable the OWASP 3.0 rule set within the web application firewall blade.

With this scenario we have good mix of integration to hybrid resources and utilising azure app service technologies.



I hope you find this useful. 



