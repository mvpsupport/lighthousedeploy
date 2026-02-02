# Azure Lighthouse Deployment Wizard

This template provides a guided wizard experience for deploying Azure Lighthouse delegations, enabling service providers to manage customer Azure subscriptions.

## What is Azure Lighthouse?

Azure Lighthouse enables cross-tenant and multi-tenant management, allowing service providers to manage multiple customer tenants with enhanced automation, scalability, and governance.

## Features

- **Guided Wizard Interface**: Step-by-step UI for configuring Lighthouse delegations
- **Multiple Authorizations**: Grant access to multiple users, groups, or service principals
- **Common RBAC Roles**: Pre-configured list of frequently used Azure roles
- **Validation**: Built-in GUID validation for tenant IDs and principal IDs
- **Conditional Access Policy**: Creates a policy that enforces MFA for Azure portal, Azure management, and Microsoft Graph while excluding MSP tenant users
- **Review Step**: Summary view before deployment

## Prerequisites

- Azure subscription where you want to delegate access
- Partner tenant ID (Directory ID)
- Object IDs of users, groups, or service principals in the partner tenant
- Appropriate permissions to deploy at subscription scope

## Deployment Options

### Option 1: Deploy via Azure Portal (Recommended)

1. Click the button below to launch the deployment wizard:

   [![Deploy to Azure](https://aka.ms/deploytoazurebutton)][deploy-link]

   [deploy-link]: https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fxc-chris%2Flighthousedeploy%2Fmain%2Fazuredeploy.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Fxc-chris%2Flighthousedeploy%2Fmain%2FcreateUiDefinition.json

2. Follow the wizard steps:
   - **Basics**: Enter delegation name and description
   - **Partner Information**: Provide the managing tenant ID
   - **Access & Permissions**: Configure authorizations (who gets what access)
   - **Review + Create**: Review and deploy

### Option 2: Deploy via Azure CLI

```bash
az deployment sub create \
  --name lighthouse-delegation \
  --location <region> \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json
```

### Option 3: Deploy via PowerShell

```powershell
New-AzSubscriptionDeployment `
  -Name lighthouse-delegation `
  -Location <region> `
  -TemplateFile .\azuredeploy.json `
  -TemplateParameterFile .\azuredeploy.parameters.json
```

## Configuration Guide

### Finding Required Information

#### Partner Tenant ID
1. Sign in to Azure portal with partner account
2. Navigate to **Microsoft Entra ID** > **Overview**
3. Copy the **Tenant ID** (also called Directory ID)

#### Principal Object IDs
1. In the partner's Azure portal, go to **Microsoft Entra ID**
2. Navigate to **Groups** (recommended) or **Users**
3. Select the group/user that should receive delegated access
4. Copy the **Object ID**

**Best Practice**: Use Azure AD groups instead of individual users for easier management.

### Choosing Roles

The wizard includes common Azure RBAC roles:

| Role | Use Case |
|------|----------|
| **Reader** | View-only access to all resources |
| **Contributor** | Manage all resources except access control |
| **Owner** | Full access including access management (use cautiously) |
| **Virtual Machine Contributor** | Manage VMs (without network/storage) |
| **Network Contributor** | Manage network resources |
| **Storage Account Contributor** | Manage storage accounts |
| **Backup Contributor** | Manage backup operations |
| **Monitoring Contributor** | Manage monitoring resources |
| **Security Admin** | View and update security policies |
| **Support Request Contributor** | Create and manage support tickets |

### Example Authorizations

**Scenario 1: MSP with tiered access**
- L1 Support Group → Reader
- L2 Engineers → Contributor
- Senior Architects → Owner (limited to specific scope)

**Scenario 2: Specialized management**
- Network Team → Network Contributor
- Backup Team → Backup Contributor
- Security Team → Security Admin

## Template Parameters

### Required Parameters

- `managedByTenantId`: Partner tenant ID (GUID)
- `mspOfferName`: Display name for the delegation
- `mspOfferDescription`: Description of the delegation
- `authorizations`: Array of authorization objects
- `conditionalAccessPolicyDisplayName`: Display name for the conditional access policy
- `createConditionalAccessPolicy`: Whether to request the conditional access policy (intent only)

### Authorization Object Structure

```json
{
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "principalIdDisplayName": "IT Operations Team",
  "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
}
```

## Post-Deployment

### Verify Deployment

1. Navigate to **Service providers** in the Azure portal
2. Verify your delegation appears under **Service provider offers**
3. Check the authorizations are correctly configured

### Partner Access

After deployment, users in the partner tenant can:

1. Sign in to Azure portal with their partner credentials
2. Use the directory switcher to access the customer subscription
3. Navigate to **My customers** to see all delegated subscriptions
4. Manage resources according to assigned roles

### Modify or Remove Delegation

**To modify**: Deploy the template again with updated parameters

**To remove**:
```bash
az managedservices assignment delete \
  --assignment <assignment-id>
```

Or delete from **Service providers** blade in the portal.

## Security Best Practices

1. **Use Groups**: Assign roles to Azure AD groups, not individual users
2. **Least Privilege**: Grant minimum permissions needed
3. **Regular Audits**: Review delegations and access logs regularly
4. **Multi-Factor Authentication**: Require MFA for partner users
5. **Conditional Access**: Apply policies that exclude MSP tenant users where appropriate
6. **Monitor Activity**: Use Azure Activity Log to track partner actions

## Troubleshooting

### Common Issues

**Error: Invalid tenant ID**
- Verify the GUID format is correct
- Ensure you're using the partner's tenant ID, not the customer's

**Error: Principal not found**
- Confirm the Object ID exists in the partner tenant
- Check that you copied the Object ID, not the Application ID

**Delegation not visible**
- Allow up to 15 minutes for Azure to propagate changes
- Refresh the Service providers blade

**Partner cannot access**
- Verify the partner user is member of the authorized group
- Check that the role assignment includes necessary permissions
- Ensure partner is switching to the correct directory

## Resources

- [Azure Lighthouse Documentation](https://docs.microsoft.com/azure/lighthouse/)
- [Azure RBAC Roles](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)
- [Lighthouse Best Practices](https://docs.microsoft.com/azure/lighthouse/concepts/recommended-security-practices)

## Support

For issues with the template, please file an issue in the repository.

For Azure Lighthouse support, contact Microsoft Azure support.
