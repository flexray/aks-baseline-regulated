# Azure AD Conditional Access

Azure AD Conditional Access supports policies that apply directly to Kubernetes cluster access. In your policy you can apply any of the standard conditions and access controls, and scope them to apply specifically for your cluster's _Azure Kubernetes Service AAD Server_ cloud app.

For example, you could require that devices accessing the API Server are being performed exclusively from devices marked as compliant, only from select or trusted locations, only from select OSes, etc. Conditional access will often then be applied when connecting to your cluster from [your jump box](./deploy/06-aks-jumpboximage.md), ensuring that the jump box itself and the user performing the action have met core conditional criteria to perform any API Server interaction.

Work with your Conditional Access administrator [to apply a policy](https://docs.microsoft.com/azure/aks/managed-aad#use-conditional-access-with-azure-ad-and-aks) that helps you achieve your access governance requirements. In addition to the portal, you can also perform the assignment via the AzureAD Windows PowerShell module.

Remember to test all conditional access policies using a safe and controlled rollout procedure before applying to all users. Paired with [Azure AD JIT access](https://docs.microsoft.com/azure/aks/managed-aad#configure-just-in-time-cluster-access-with-azure-ad-and-aks), this provides a very robust access control solution for your private cluster.

## Applying via Windows PowerShell

For many administrators, PowerShell is already an understood scripting tool. The following example shows how to use the Azure AD PowerShell module to apply a Conditional Access policy.

```powershell
Install-Module -Name AzureAD -Force -Scope CurrentUser

# Must see AzureAD listed at a version >= 2.0.2.106
Get-InstalledModule -Name AzureAD

Connect-AzureAD -TenantId <your-tenant-guid>

$conditions = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessConditionSet
$conditions.Applications = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessApplicationCondition
$conditions.Applications.IncludeApplications = "<your-cluster's-server-app-guid>"
$conditions.Users = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessUserCondition
$conditions.Users.IncludeUsers = "All" # Or do per-group policies based on risk profile of those groups.
# Additional $conditions as desired

$controls = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessGrantControls
# Configure $controls as desired

New-AzureADMSConditionalAccessPolicy -DisplayName "AKS API Server <server name> Access Policy" -State "on" -Conditions $conditions -GrantControls $controls
```

For more examples, see [Configure Conditional Access policies using Azure AD PowerShell](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/01-configure/powershell)

### Alternatives to Windows PowerShell

Azure AD conditional access policies can be managed in the following ways if Windows PowerShell is not aligned with your preferred toolset.

* Within the Azure AD portal directly
* [Microsoft Graph API](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/01-configure/graphapi), including advanced flows like [using Logic Apps to facilitate deployment](https://github.com/Azure-Samples/azure-ad-conditional-access-apis/tree/main/01-configure/templates)

## Next Steps

* See additional [Authentication & Authorization considerations](./additional-considerations.md#authentication--authorization)
* [Prep for Azure AD Integration](./deploy/03-aad.md)
