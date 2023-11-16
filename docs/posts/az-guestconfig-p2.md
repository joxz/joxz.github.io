---
date: 2023-07-07
description: "Azure Policy Guest Configuration for Linux - Part 2"
categories:
  - lab

tags:
  - azure-policy
  - vm-extensions
  - guestconfiguration
  - dsc
  - linux

comments: true
authors: [jo]
---

# Azure Policy Guest Configuration for Linux - Part 2

Continuation of the [first part](az-guestconfig-p1.md/){target="_blank"} of this series. The Guest Configuration Package is now ready to be tested and deployed!
<!-- more -->

Make sure you're signed in to Azure in the dev container `pwsh` session for this part:

```pwsh
➜ Connect-AzAccount -DeviceCode
➜ Set-AzContext -subscription {subscriptionname}
```

## Test and Verify Package

Package can be tested with:

```pwsh
➜ Get-GuestConfigurationPackageComplianceStatus -path './output/GCPackages/GCFilePresent_0.0.1.zip'
                                                                                                                        
additionalProperties : {}
assignmentName       : GCFilePresent
complianceStatus     : False
endTime              : 7/3/2023 8:24:59 PM
jobId                : f9d51551-22d6-4743-92ad-1ced0021cca1
operationtype        : Consistency
resources            : {@{complianceStatus=False; properties=; reasons=System.Object[]}}
startTime            : 7/3/2023 8:24:32 PM
```

`complianceStatus` should equal `False` on a new system

Test remediation:

```pwsh
➜ Start-GuestConfigurationPackageRemediation -path './output/GCPackages/GCFilePresent_0.0.1.zip'
                                                                                                                        
additionalProperties : {}
assignmentName       : GCFilePresent
complianceStatus     : True
endTime              : 7/3/2023 8:25:59 PM
jobId                : dcdb25e6-e760-4e7b-8e1a-f33c8ece59d5
operationtype        : Consistency
resources            : {@{complianceStatus=True; properties=; reasons=System.Object[]}}
startTime            : 7/3/2023 8:25:26 PM
```

`complianceStatus` is reported back as `True` and can be double checked with the same `Get-GuestConfigurationPackageComplianceStatus` as above. Also, check for the file in `/tmp`:

```pwsh
➜ cat /tmp/00dummy.txt
TESTTESTTESTTESTTESTTESTTESTTESTTESTTESTTESTTEST
```

## Publishing the Package

To publish the package, a storage account is needed ([:octicons-link-external-16: communication via private endpoints is supported](https://learn.microsoft.com/en-us/azure/governance/machine-configuration/overview#communicate-over-private-link-in-azure){target="_blank"}), and the package must be uploaded to blob storage. Please follow the steps outlined [:octicons-link-external-16: here](https://learn.microsoft.com/en-us/azure/governance/machine-configuration/how-to-publish-package#publish-a-configuration-package){target="_blank"} on how to do that.

The resulting blob URL stored in `$contentUri` you need for the next step should look like this:

```pwsh
URL template:
https://{storageaccountname}.blob.core.windows.net/{containername}/{filename}?{sas-token}

Real URL:
https://gcpolpackages46282.blob.core.windows.net/guestconfiguration/GCFilePresent_0.0.1.zip?sv=2022-11-02&ss=bfqt&srt=sco&sp=rwdlacupiytfx&se=2023-07-22T14:31:11Z&st=2023-07-03T06:31:11Z&spr=https&sig=QwD2COOrgdsjbkteKau00%2B5duy4sbkT5JSykQp21ke8%3D

➜ $contenturi
https://gcpolpackages46282.blob.core.windows.net/guestconfiguration/GCFilePresent_0.0.1.zip?sv=2022-11-02&ss=bfqt&srt=sco&sp=rwdlacupiytfx&se=2023-07-22T14:31:11Z&st=2023-07-03T06:31:11Z&spr=https&sig=QwD2COOrgdsjbkteKau00%2B5duy4sbkT5JSykQp21ke8%3D
```
!!! warning
    SAS token lifetime, even if it is set to 3 years in the docs, is something that needs to be tracked and taken care of manually

Please make sure the `$contentUri` variable is set correctly

## Create a Policy Definition

Detailed explanations for this step are available [:octicons-link-external-16: here](https://learn.microsoft.com/en-us/azure/governance/machine-configuration/how-to-create-policy-definition#create-an-azure-policy-definition){target="_blank"}

```pwsh
➜ $policyid = $(new-guid)

➜ $PolicyConfig      = @{
    PolicyId      = $policyid
    ContentUri    = $contenturi
    DisplayName   = 'GCFilePresent'
    Description   = 'Ensures a specific file is present'
    Path          = './policies/'
    Platform      = 'Linux'
    PolicyVersion = '1.0.0'
    Mode          = 'ApplyAndAutoCorrect'
  }

➜ New-GuestConfigurationPolicy @PolicyConfig

Name          Path                                                                                                                PolicyId
----          ----                                                                                                                --------
GCFilePresent /workspaces/guestconfiguration-linux/guestconfiguration/GCFilePresent/policies/GCFilePresent_DeployIfNotExists.json d0403c9a-42c0-4df0-a125-41ca046987e4
```

This will create a policy definition in `policies/GCFilePresent_DeployIfNotExists.json`

!!! info
    `Mode = 'ApplyAndAutoCorrect'` creates a `DeployIfNotExists` policy. That means, the policy needs to create a managed identity with the assignment so it has the permissions (`Guest Configuration Resource Contributor`) to create remediation tasks

!!! note
    [:octicons-link-external-16: Using parameters in custom machine configuration policy definitions](https://learn.microsoft.com/en-us/azure/governance/machine-configuration/how-to-create-policy-definition#using-parameters-in-custom-machine-configuration-policy-definitions){target="_blank"} did not work for me, it would always require the `ResourcePropertyValue` parameter. Setting this parameter would kill the whole parameterization idea. This is probably an error in the module, or I'm missing/overseeing something...

## Register Policy Definition

```pwsh
➜ New-AzPolicyDefinition -Name 'GC Ensures file is present' -Policy '.\policies\GCFilePresent_DeployIfNotExists.json'

Name               : GC Ensures file is present
ResourceId         : /subscriptions/{subscriptionid}/providers/Microsoft.Authorization/policyDefinitions/GC Ensures file is present
ResourceName       : GC Ensures file is present
ResourceType       : Microsoft.Authorization/policyDefinitions
SubscriptionId     : {subscriptionid}
Properties         : Microsoft.Azure.Commands.ResourceManager.Cmdlets.Implementation.Policy.PsPolicyDefinitionProperties
PolicyDefinitionId : /subscriptions/{subscriptionid}/providers/Microsoft.Authorization/policyDefinitions/GC Ensures file is present
```

The policy is now ready to be assigned!

In the next part we will take a look at the policy dashboard, VM agent, how fast remediation sets in and test everything on a Linux VM

## More Resources

- [:octicons-link-external-16: Policy Lifecycle](https://learn.microsoft.com/en-us/azure/governance/machine-configuration/how-to-create-policy-definition#policy-lifecycle){target="_blank"}
- [:octicons-link-external-16: Remix with a Twist: 7 steps to author, develop, and deploy custom recommendations for Windows using Guest Configuration](https://swiftsolves.substack.com/p/remix-with-a-twist-7-steps-to-author){target="_blank"}
- [:octicons-link-external-16: Azure Governance: how to control system configurations in hybrid and multicloud environments](https://francescomolfese.it/en/2021/04/azure-governance-come-controllare-le-configurazioni-dei-sistemi-in-ambienti-ibridi-e-multicloud/){target="_blank"}
