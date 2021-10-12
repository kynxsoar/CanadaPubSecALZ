# Onboarding Guide for Azure DevOps

This document provides steps required to onboard to the Azure Landing Zones design based on Azure DevOps Pipelines.

**All steps will need to be repeated per Azure AD tenant.**

## Step 1:  Create Service Principal Account & Assign RBAC

A service principal account is required to automate the Azure DevOps pipelines. 

* **Service Principal Name**:  any name (i.e. spn-azure-platform-ops)

* **RBAC Assignment**

    * Scope:  Tenant Root Group (this is a management group)

    * Role:  Owner

## Step 2:  Configure Service Connection in Azure DevOps Project Configuration

* **Scope Level**:  Management Group

* **Service Connection Name**:  spn-azure-platform-ops

    *Service Connection Name will be used to configure Azure DevOps Pipelines.*

*  **Instructions**:  [Service connections in Azure Pipelines - Azure Pipelines | Microsoft Docs](https://docs.microsoft.com/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml)


## Step 3:  Configure Management Group Deployment

### Step: 3.1: Update common.yml in git repository

Create/edit **./config/variables/common.yml** in Git.  This file is used in all Azure DevOps pipelines.

**Sample YAML**
```yaml
variables:

  deploymentRegion: canadacentral
  serviceConnection: spn-azure-platform-ops
  vmImage: ubuntu-latest
  deployOperation: create

```

### Step 3.2:  Update environment config file in git repository

1. Create/edit **./config/variables/<devops-org-name>-<branch-name>.yml** in Git (i.e. CanadaESLZ-main.yml).  This file name is automatically inferred based on the Azure DevOps organization name and the branch.

    **Sample environment YAML**

    ```yaml
    variables:

        # Management Groups
        var-parentManagementGroupId: abcddfdb-bef5-46d9-99cf-ed67dabc8783
        var-topLevelManagementGroupName: pubsec

    ```

2. Commit the changes to git repository

### Step 3.3:  Configure Azure DevOps Pipeline

1. Pipeline definition for Management Group.

    *Note: Pipelines are stored as YAML definitions in Git and imported into Azure DevOps Pipelines.  This approach allows for portability and change tracking.*

    1.    Go to Pipelines
    2.    New Pipeline
    3.    Choose Azure Repos Git
    4.    Select Repository
    5.    Select Existing Azure Pipeline YAML file
    6.    Identify the pipeline in `.pipelines/management-groups.yml`.
    7.  Save the pipeline (don't run it yet)
    8.  Rename the pipeline to `management-groups-ci`


2. Run pipeline and wait for completion.


## Step 4:  Logging Landing Zone

### Step 4.1:  Setup Azure AD Security Group (Optional)

At least one Azure AD Security Group is required for role assignment.  Role assignment can be set for Owner, Contributor, and/or Reader roles.  Note down the Security Group object id, it will be required for next step.

### Step 4.2:  Update configuration files in git repository

Set the configuration parameters even if there's an existing central Log Analytics Workspace.  These settings are used by other deployments such as Azure Policy for Log Analytics.  In this case, use the values of the existing Log Analytics Workspace.

When a Log Analytics Workspace & Automation account already exists, enter Subscription ID, Resource Group, Log Analytics Workspace name and Automation account name.  The automation will update the existing deployment instead of creating new resources.

1. Edit `./config/variables/<devops-org-name>-<branch-name>.yml` in Git.  This configuration file was created in Step 3.

    **Sample environment YAML**

    ```yml
        variables:
            # Management Groups
            var-parentManagementGroupId: 343ddfdb-bef5-46d9-99cf-ed67d5948783
            var-topLevelManagementGroupName: pubsec

            # Logging
            var-logging-managementGroupId: pubsecPlatform
            var-logging-subscriptionId: bc0a4f9f-07fa-4284-b1bd-fbad38578d3a
            var-logging-logAnalyticsResourceGroupName: pubsec-central-logging-rg
            var-logging-logAnalyticsWorkspaceName: log-analytics-workspace
            var-logging-logAnalyticsAutomationAccountName: automation-account
            var-logging-diagnosticSettingsforNetworkSecurityGroupsStoragePrefix: pubsecnsg
            var-logging-serviceHealthAlerts: >
                {
                    "resourceGroupName": "pubsec-service-health",
                    "incidentTypes": [ "Incident", "Security" ],
                    "regions": [ "Global", "Canada East", "Canada Central" ],
                    "receivers": {
                        "app": [ "alzcanadapubsec@microsoft.com" ],
                        "email": [ "alzcanadapubsec@microsoft.com" ],
                        "sms": [
                            { "countryCode": "1", "phoneNumber": "5555555555" }
                        ],
                        "voice": [
                            { "countryCode": "1", "phoneNumber": "5555555555" }
                        ]
                    }
                }
            var-logging-securityCenter: >
                {
                    "email": "alzcanadapubsec@microsoft.com",
                    "phone": "5555555555"
                }
            var-logging-subscriptionRoleAssignments: >
                [
                    {
                        "comments": "Built-in Contributor Role",
                        "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c",
                        "securityGroupObjectIds": [
                            "38f33f7e-a471-4630-8ce9-c6653495a2ee"
                        ]
                    }
                ]
            var-logging-subscriptionBudget: >
                {
                    "createBudget": false,
                    "name": "MonthlySubscriptionBudget",
                    "amount": 1000,
                    "timeGrain": "Monthly",
                    "contactEmails": [ "alzcanadapubsec@microsoft.com" ]
                }
            var-logging-subscriptionTags: >
                {
                    "ISSO": "isso-tbd"
                }
            var-logging-resourceTags: >
                {
                    "ClientOrganization": "client-organization-tag",
                    "CostCenter": "cost-center-tag",
                    "DataSensitivity": "data-sensitivity-tag",
                    "ProjectContact": "project-contact-tag",
                    "ProjectName": "project-name-tag",
                    "TechnicalContact": "technical-contact-tag"
                }
    ```

2. Commit the changes to git repository.

### Step 4.3:  Configure Azure DevOps Pipeline (only required if a new central Log Analytics Workspace is required)

1. Pipeline definition for Central Logging.

    *Note: Pipelines are stored as YAML definitions in Git and imported into Azure DevOps Pipelines.  This approach allows for portability and change tracking.*

    1.    Go to Pipelines
    2.    New Pipeline
    3.    Choose Azure Repos Git
    4.    Select Repository
    5.    Select Existing Azure Pipeline YAML file
    6.    Identify the pipeline in `.pipelines/platform-logging.yml`.
    7.  Save the pipeline (don't run it yet)
    8.  Rename the pipeline to `platform-logging-ci`


2. Run pipeline and wait for completion.

## Step 5:  Configure Azure Policies

1. Pipeline definition for Azure Policies.

    *Note: Pipelines are stored as YAML definitions in Git and imported into Azure DevOps Pipelines.  This approach allows for portability and change tracking.*

    1.    Go to Pipelines
    2.    New Pipeline
    3.    Choose Azure Repos Git
    4.    Select Repository
    5.    Select Existing Azure Pipeline YAML file
    6.    Identify the pipeline in `.pipelines/policy.yml`.
    7.  Save the pipeline (don't run it yet)
    8.  Rename the pipeline to `policy-ci`


2. Run pipeline and wait for completion.


## Step 6:  Configure Custom Roles

1. Pipeline definition for Custom Roles.

    *Note: Pipelines are stored as YAML definitions in Git and imported into Azure DevOps Pipelines.  This approach allows for portability and change tracking.*

    1.    Go to Pipelines
    2.    New Pipeline
    3.    Choose Azure Repos Git
    4.    Select Repository
    5.    Select Existing Azure Pipeline YAML file
    6.    Identify the pipeline in `.pipelines/roles.yml`.
    7.  Save the pipeline (don't run it yet)
    8.  Rename the pipeline to `roles-ci`


2. Run pipeline and wait for completion.

## Step 7:  Configure Hub Networking using NVAs

1. Edit `./config/variables/<devops-org-name>-<branch-name>.yml` in Git.  This configuration file was created in Step 3.

   Update configuration with the networking section.  There are two options for Hub Networking:

    1. Hub Networking with Azure Firewall
    2. Hub Networking with Fortinet Firewall (NVA)

    Depending on the preference, you may delete/comment the configuration that is not required.

    **Sample environment YAML**

    ```yml
        variables:
            # Management Groups
            var-parentManagementGroupId: 343ddfdb-bef5-46d9-99cf-ed67d5948783
            var-topLevelManagementGroupName: pubsec

            # Logging
            var-logging-managementGroupId: pubsecPlatform
            var-logging-subscriptionId: bc0a4f9f-07fa-4284-b1bd-fbad38578d3a
            var-logging-logAnalyticsResourceGroupName: pubsec-central-logging-rg
            var-logging-logAnalyticsWorkspaceName: log-analytics-workspace
            var-logging-logAnalyticsAutomationAccountName: automation-account
            var-logging-diagnosticSettingsforNetworkSecurityGroupsStoragePrefix: pubsecnsg
            var-logging-serviceHealthAlerts: >
                {
                    "resourceGroupName": "pubsec-service-health",
                    "incidentTypes": [ "Incident", "Security" ],
                    "regions": [ "Global", "Canada East", "Canada Central" ],
                    "receivers": {
                        "app": [ "alzcanadapubsec@microsoft.com" ],
                        "email": [ "alzcanadapubsec@microsoft.com" ],
                        "sms": [
                            { "countryCode": "1", "phoneNumber": "5555555555" }
                        ],
                        "voice": [
                            { "countryCode": "1", "phoneNumber": "5555555555" }
                        ]
                    }
                }
            var-logging-securityCenter: >
                {
                    "email": "alzcanadapubsec@microsoft.com",
                    "phone": "5555555555"
                }
            var-logging-subscriptionRoleAssignments: >
                [
                    {
                        "comments": "Built-in Contributor Role",
                        "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c",
                        "securityGroupObjectIds": [
                            "38f33f7e-a471-4630-8ce9-c6653495a2ee"
                        ]
                    }
                ]
            var-logging-subscriptionBudget: >
                {
                    "createBudget": false,
                    "name": "MonthlySubscriptionBudget",
                    "amount": 1000,
                    "timeGrain": "Monthly",
                    "contactEmails": [ "alzcanadapubsec@microsoft.com" ]
                }
            var-logging-subscriptionTags: >
                {
                    "ISSO": "isso-tbd"
                }
            var-logging-resourceTags: >
                {
                    "ClientOrganization": "client-organization-tag",
                    "CostCenter": "cost-center-tag",
                    "DataSensitivity": "data-sensitivity-tag",
                    "ProjectContact": "project-contact-tag",
                    "ProjectName": "project-name-tag",
                    "TechnicalContact": "technical-contact-tag"
                }

            # Hub Networking
            var-hubnetwork-managementGroupId: pubsecPlatform
            var-hubnetwork-subscriptionId: ed7f4eed-9010-4227-b115-2a5e37728f27
            var-hubnetwork-serviceHealthAlerts: >
                {
                    "resourceGroupName": "pubsec-service-health",
                    "incidentTypes": [ "Incident", "Security" ],
                    "regions": [ "Global", "Canada East", "Canada Central" ],
                    "receivers": {
                        "app": [ "alzcanadapubsec@microsoft.com" ],
                        "email": [ "alzcanadapubsec@microsoft.com" ],
                        "sms": [
                            { "countryCode": "1", "phoneNumber": "5555555555" }
                        ],
                        "voice": [
                            { "countryCode": "1", "phoneNumber": "5555555555" }
                        ]
                    }
                }
            var-hubnetwork-securityCenter: >
                {
                    "email": "alzcanadapubsec@microsoft.com",
                    "phone": "5555555555"
                }
            var-hubnetwork-subscriptionRoleAssignments: >
                [
                {
                    "comments": "Built-in Contributor Role",
                    "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c",
                    "securityGroupObjectIds": [
                        "38f33f7e-a471-4630-8ce9-c6653495a2ee"
                    ]
                }
                ]
            var-hubnetwork-subscriptionBudget: >
                {
                    "createBudget": false,
                    "name": "MonthlySubscriptionBudget",
                    "amount": 1000,
                    "timeGrain": "Monthly",
                    "contactEmails": [ "alzcanadapubsec@microsoft.com" ]
                }
            var-hubnetwork-subscriptionTags: >
                {
                    "ISSO": "isso-tbd"
                }
            var-hubnetwork-resourceTags: >
                {
                    "ClientOrganization": "client-organization-tag",
                    "CostCenter": "cost-center-tag",
                    "DataSensitivity": "data-sensitivity-tag",
                    "ProjectContact": "project-contact-tag",
                    "ProjectName": "project-name-tag",
                    "TechnicalContact": "technical-contact-tag"
                }

            ## Hub Networking - Private Dns Zones
            var-hubnetwork-deployPrivateDnsZones: true
            var-hubnetwork-rgPrivateDnsZonesName: pubsec-dns-rg

            ## Hub Networking - DDOS
            var-hubnetwork-deployDdosStandard: false
            var-hubnetwork-rgDdosName: pubsec-ddos-rg
            var-hubnetwork-ddosPlanName: ddos-plan

            ## Hub Networking - Public Zone
            var-hubnetwork-rgPazName: pubsec-public-access-zone-rg

            ## Hub Networking - Management Restricted Zone Virtual Network
            var-hubnetwork-rgMrzName: pubsec-management-restricted-zone-rg
            var-hubnetwork-mrzVnetName: management-restricted-vnet
            var-hubnetwork-mrzVnetAddressPrefixRFC1918: 10.18.4.0/22

            var-hubnetwork-mrzMazSubnetName: MazSubnet
            var-hubnetwork-mrzMazSubnetAddressPrefix: 10.18.4.0/25
            
            var-hubnetwork-mrzInfSubnetName: InfSubnet
            var-hubnetwork-mrzInfSubnetAddressPrefix: 10.18.4.128/25
            
            var-hubnetwork-mrzSecSubnetName: SecSubnet
            var-hubnetwork-mrzSecSubnetAddressPrefix: 10.18.5.0/26
            
            var-hubnetwork-mrzLogSubnetName: LogSubnet
            var-hubnetwork-mrzLogSubnetAddressPrefix: 10.18.5.64/26
            
            var-hubnetwork-mrzMgmtSubnetName: MgmtSubnet
            var-hubnetwork-mrzMgmtSubnetAddressPrefix: 10.18.5.128/26

            var-hubnetwork-bastionName: bastion

            ####################################################################################
            ### Hub Networking with Azure Firewall                                           ###
            ####################################################################################
            var-hubnetwork-azfw-rgPolicyName: pubsec-azure-firewall-policy-rg
            var-hubnetwork-azfw-policyName: pubsecAzureFirewallPolicy
            
            var-hubnetwork-azfw-rgHubName: pubsec-hub-networking-rg
            var-hubnetwork-azfw-hubVnetName: hub-vnet
            var-hubnetwork-azfw-hubVnetAddressPrefixRFC1918: 10.18.0.0/22
            var-hubnetwork-azfw-hubVnetAddressPrefixRFC6598: 100.60.0.0/16
            var-hubnetwork-azfw-hubVnetAddressPrefixBastion: 192.168.0.0/16

            var-hubnetwork-azfw-hubPazSubnetName: PAZSubnet
            var-hubnetwork-azfw-hubPazSubnetAddressPrefix: 100.60.1.0/24

            var-hubnetwork-azfw-hubGatewaySubnetPrefix: 10.18.0.0/27
            var-hubnetwork-azfw-hubAzureFirewallSubnetAddressPrefix: 10.18.1.0/24
            var-hubnetwork-azfw-hubAzureFirewallManagementSubnetAddressPrefix: 10.18.2.0/26
            var-hubnetwork-azfw-hubBastionSubnetAddressPrefix: 192.168.0.0/24
            
            var-hubnetwork-azfw-azureFirewallName: pubsecAzureFirewall
            var-hubnetwork-azfw-azureFirewallZones: '["1", "2", "3"]'
            var-hubnetwork-azfw-azureFirewallForcedTunnelingEnabled: false
            var-hubnetwork-azfw-azureFirewallForcedTunnelingNextHop: 10.17.1.4

            ####################################################################################
            ### Hub Networking with Fortinet Firewalls                                       ###
            ####################################################################################
            
            ## Hub Networking - Core Virtual Network
            var-hubnetwork-nva-rgHubName: pubsec-hub-networking-rg
            var-hubnetwork-nva-hubVnetName: hub-vnet
            var-hubnetwork-nva-hubVnetAddressPrefixRFC1918: 10.18.0.0/22
            var-hubnetwork-nva-hubVnetAddressPrefixRFC6598: 100.60.0.0/16
            var-hubnetwork-nva-hubVnetAddressPrefixBastion: 192.168.0.0/16

            var-hubnetwork-nva-hubEanSubnetName: EanSubnet
            var-hubnetwork-nva-hubEanSubnetAddressPrefix: 10.18.0.0/27

            var-hubnetwork-nva-hubPublicSubnetName: PublicSubnet
            var-hubnetwork-nva-hubPublicSubnetAddressPrefix: 100.60.0.0/24

            var-hubnetwork-nva-hubPazSubnetName: PAZSubnet
            var-hubnetwork-nva-hubPazSubnetAddressPrefix: 100.60.1.0/24

            var-hubnetwork-nva-hubDevIntSubnetName: DevIntSubnet
            var-hubnetwork-nva-hubDevIntSubnetAddressPrefix: 10.18.0.64/27

            var-hubnetwork-nva-hubProdIntSubnetName: PrdIntSubnet
            var-hubnetwork-nva-hubProdIntSubnetAddressPrefix: 10.18.0.32/27

            var-hubnetwork-nva-hubMrzIntSubnetName: MrzSubnet
            var-hubnetwork-nva-hubMrzIntSubnetAddressPrefix: 10.18.0.96/27

            var-hubnetwork-nva-hubHASubnetName: HASubnet
            var-hubnetwork-nva-hubHASubnetAddressPrefix: 10.18.0.128/28

            var-hubnetwork-nva-hubGatewaySubnetPrefix: 10.18.1.0/27

            var-hubnetwork-nva-hubBastionSubnetAddressPrefix: 192.168.0.0/24

            ## Hub Networking - Firewall Virtual Appliances
            var-hubnetwork-nva-deployFirewallVMs: false
            var-hubnetwork-nva-useFortigateFW: false

            ### Hub Networking - Firewall Virtual Appliances - For Non-production Traffic
            var-hubnetwork-nva-fwDevILBName: pubsecDevFWILB
            var-hubnetwork-nva-fwDevVMSku: Standard_D8s_v4
            var-hubnetwork-nva-fwDevVM1Name: pubsecDevFW1
            var-hubnetwork-nva-fwDevVM2Name: pubsecDevFW2
            var-hubnetwork-nva-fwDevILBExternalFacingIP: 100.60.0.7
            var-hubnetwork-nva-fwDevVM1ExternalFacingIP: 100.60.0.8
            var-hubnetwork-nva-fwDevVM2ExternalFacingIP: 100.60.0.9
            var-hubnetwork-nva-fwDevILBMrzIntIP: 10.18.0.103
            var-hubnetwork-nva-fwDevVM1MrzIntIP: 10.18.0.104
            var-hubnetwork-nva-fwDevVM2MrzIntIP: 10.18.0.105
            var-hubnetwork-nva-fwDevILBDevIntIP: 10.18.0.68
            var-hubnetwork-nva-fwDevVM1DevIntIP: 10.18.0.69
            var-hubnetwork-nva-fwDevVM2DevIntIP: 10.18.0.70
            var-hubnetwork-nva-fwDevVM1HAIP: 10.18.0.134
            var-hubnetwork-nva-fwDevVM2HAIP: 10.18.0.135

            ### Hub Networking - Firewall Virtual Appliances - For Production Traffic
            var-hubnetwork-nva-fwProdILBName: pubsecProdFWILB
            var-hubnetwork-nva-fwProdVMSku: Standard_F8s_v2
            var-hubnetwork-nva-fwProdVM1Name: pubsecProdFW1
            var-hubnetwork-nva-fwProdVM2Name: pubsecProdFW2
            var-hubnetwork-nva-fwProdILBExternalFacingIP: 100.60.0.4
            var-hubnetwork-nva-fwProdVM1ExternalFacingIP: 100.60.0.5
            var-hubnetwork-nva-fwProdVM2ExternalFacingIP: 100.60.0.6
            var-hubnetwork-nva-fwProdILBMrzIntIP: 10.18.0.100
            var-hubnetwork-nva-fwProdVM1MrzIntIP: 10.18.0.101
            var-hubnetwork-nva-fwProdVM2MrzIntIP: 10.18.0.102
            var-hubnetwork-nva-fwProdILBPrdIntIP: 10.18.0.36
            var-hubnetwork-nva-fwProdVM1PrdIntIP: 10.18.0.37
            var-hubnetwork-nva-fwProdVM2PrdIntIP: 10.18.0.38
            var-hubnetwork-nva-fwProdVM1HAIP: 10.18.0.132
            var-hubnetwork-nva-fwProdVM2HAIP: 10.18.0.133
    ```

2. Configure Variable Group:  firewall-secrets **(required for Fortinet Firewall deployment)**

    * In Azure DevOps, go to Pipelines -> Library
    * Select + Variable group
    * Set Variable group name:  firewall-secrets
    * Add two variables:

      These two variables are used when creating Firewall virtual machines.  These are temporary passwords and recommended to be changed after creation. The same username and password are used for all virtual machines.

      When creating both variables, toggle the lock icon to make it a secret.  This ensures that the values are not shown in logs nor to Azure DevOps users.

      Write down the username and password as it's not retrievable once saved.

        * var-hubnetwork-nva-fwUsername
        * var-hubnetwork-nva-fwPassword

    * Click Save 

3. Configure Pipeline for Platform – Hub Networking using Azure Firewall (only if Azure Firewall based Hub Networking is used)

    > Note: Pipelines are stored as YAML definitions in Git and imported into Azure DevOps Pipelines.  This approach allows for portability and change tracking.

    1. Go to Pipelines

    2. New Pipeline

        1. Choose Azure Repos Git
        2. Select Repository
        3. Select Existing Azure Pipeline YAML file
        4. Identify the pipeline in `.pipelines/platform-connectivity-hub-azfw-policy.yml`.
        6. Save the pipeline (don't run it yet)
        7. Rename the pipeline to `platform-connectivity-hub-azfw-policy-ci`

    3. New Pipeline
    
        1. Choose Azure Repos Git
        2. Select Repository
        3. Select Existing Azure Pipeline YAML file
        4. Identify the pipeline in `.pipelines/platform-connectivity-hub-azfw.yml`.
        6. Save the pipeline (don't run it yet)
        7. Rename the pipeline to `platform-connectivity-hub-azfw-ci`


4. Configure Pipeline for Platform – Hub Networking using NVAs (only if Fortinet Firewall based Hub Networking is used)

    > Note: Pipelines are stored as YAML definitions in Git and imported into Azure DevOps Pipelines.  This approach allows for portability and change tracking.

    1. Go to Pipelines
    2. New Pipeline
    3. Choose Azure Repos Git
    4. Select Repository
    5. Select Existing Azure Pipeline YAML file
    6. Identify the pipeline in `.pipelines/platform-connectivity-hub-nva.yml`.
    7. Save the pipeline (don't run it yet)
    8. Rename the pipeline to `platform-connectivity-hub-nva-ci`

5. If using Fortinet Firewalls, configure Pipeline permissions for the secrets.

    * In Azure DevOps, go to Pipelines -> Library
    * Select variable group previously created (i.e. "firewall-secrets")
    * Click "Pipeline Permissions", and in resulting dialog window:
        * Click "Restrict permission"
        * Click "+" button
        * Select the "platform-connectivity-hub-nva-ci" pipeline
        * Close the dialog window

6. Run pipeline and wait for completion.

    * When using Hub Networking with Azure Firewall, run `platform-connectivity-hub-azfw-policy-ci` pipeline first.  This ensures that the Azure Firewall Policy is deployed and can be used as a reference for Azure Firewall.  This approach allows for Azure Firewall Policies (such as allow/deny rules) to be managed independently from the Hub Networking components.

## Step 8:  Configure Subscription Archetype

1. Configure Pipeline definition for subscription archetypes

    > Pipelines are stored as YAML definitions in Git and imported into Azure DevOps Pipelines.  This approach allows for portability and change tracking.

    1. Go to Pipelines
    2. New Pipeline
    3. Choose Azure Repos Git
    4. Select Repository
    5. Select Existing Azure Pipeline YAML file
    6. Identify the pipeline in `.pipelines/subscriptions.yml`.
    7. Save the pipeline (don't run it yet)
    8. Rename the pipeline to `subscription-ci`

2. Create a subscription configuration file (JSON)

    1. Make a copy of an existing subscription configuration file under `config/subscriptions/CanadaESLZ-main` as a starting point

    2. Be sure to rename the file in one of the following formats:
       * `[GUID]_[TYPE].json`
       * `[GUID]_[TYPE]_[LOCATION].json`

       Replace `[GUID]` with the subscription GUID. Replace `[TYPE]` with the subscription archetype. Optionally, add (replace) `[LOCATION]` with an Azure deployment location, e.g. `canadacentral`. If you do not specify a location in the configuration file name, the `deploymentRegion` variable will be used by default.

       > If a fourth specifier is added to the configuration filename at some future point and you do not want to supply an explicit deployment location in the third part of the configuration file name, you can either leave it empty (two consecutive underscore characters) or provide the case-sensitive value `default` to signal the `deploymentRegion` variable value should be used.

    3. Save the subscription configuration file in a subfolder (under `config/subscriptions`) that is named for your Azure DevOps organization combined with the branch name corresponding to your deployment environment. For example, if your Azure DevOps organization name is `Contoso` and your Azure Repos branch for the target deployment environment is `main`, then the subfolder name would be `Contoso-main`.

    4. Update the contents of the newly created subscription configuration file to match your deployment environment.

    5. Commit the subscription file to Azure Repos.

3. Run the subscription pipeline

    1. In Azure DevOps, go to Pipelines
    2. Select the `subscription-ci` pipeline and run it.

       > The `subscription-ci` pipeline YAML is configured, by default, to **not** run automatically; you can change this if desired.

    3. In the Run Pipelines dialog window, enter the first 4 digits of your new subscription configuration file name (4 is usually enough of the GUID to uniquely identify the subscription) between the square brackets in the `subscriptions` parameter field. For example: `[802e]`.

    4. In the Run Pipelines dialog window, click the `Run` button to start the pipeline.
