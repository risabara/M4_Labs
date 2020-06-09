
# Lab - ARM Templates - Creating Templates From Scratch


This is an open to interpretation lab where you're encouraged to think outside the box and create basic or complex templates to show a proof of concept into the different template architecture strategies:

Modular, Generic, Full Solution, Business Strategy, and Hybrid

Examples:
* Modular:
  * Create template for each resource type - NIC Card, VNet, Subnet, VM
     * Use a Master Template which uses linked templates to deploy each one
  * Design a baseline environment that'll deploy resources that are guaranteed to be in each subscription
     * Storage account for diagnostics
     * Log Analytics workspace
     * Network Watcher enablement for regions
     * Public IP Prefix
     * Virtual Network	
  * Generic - use an Object parameter or multiple parameters that ensure most of the 'heavy lifting' comes from the Parameter file instead of the Template
     * Virtual Network
  * Full Solution
     * Incorporate everything needed for a solution such as a 2-tier application solution (Frontend / Backend)
         * PaaS Web Server with a PaaS Database server

Template examples that are a very wide variety can be found within GitHub's quickstart templates -  [https://github.com/Azure/azure-quickstart-templates](https://github.com/Azure/azure-quickstart-templates)

Template strategies should consider using Nested and Linked deployments as they provide an additional layer of granularity.

If you need ideas, speak with the instructor or take the time to help whiteboard what you'd want a template to do in theory.

>[!ALERT]Please note that the lab environment is restricted by policy. You will be able to create only the Azure resources required by the lab. In particular, you will be able to create only the following kinds of resources:
>
> - Microsoft.Network
> - Microsoft.Storage
> - Micrsoft.Web
> - Micrsoft.Sql
> - Microsft.LogAnalytics 
> - Microsoft.OperationalInsights
> - Microsoft.OperationsManagement
> - Microsoft.Automation/automationAccounts
>
> You will not be able to create any Azure virtual machines of any kind.
