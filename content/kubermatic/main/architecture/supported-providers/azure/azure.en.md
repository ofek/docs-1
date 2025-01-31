+++
title = "Azure"
date = 2019-05-19T12:07:15+02:00
weight = 7

+++

## Prepare Azure Environment

For provisioning Kubernetes clusters with the [Azure cloud provider](https://github.com/kubermatic/machine-controller/tree/main/pkg/cloudprovider/provider/azure) Kubermatic Kubernetes Platform (KKP) needs a service account. Please follow the following steps steps to create a matching service account and the roles:

### Login to Azure and Get Basic Information

Login to Azure with [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) `az`.

```bash
az login
```

This command will open in your default browser a window where you can authenticate. After you successful logged in get your subscription ID.

```bash
az account show --query id -o json
```

Create a role that is used by the service account.

```
az role definition create --role-definition '{
    "Name": "Kubermatic",
    "Description": "Manage VM and Networks as well to manage Resource Groups and Tags",
    "Actions": [
          "Microsoft.Compute/*",
          "Microsoft.Network/*",
          "Microsoft.Resources/*"
    ],
    "DataActions": [],
    "NotDataActions": [],
    "AssignableScopes": ["/subscriptions/<<YOUR_SUBSCRIPTION_ID>>"]
}'

```

Get your Tenant ID

```bash
az account show --query tenantId -o json
```

create a new app with

```bash
az ad sp create-for-rbac --role="Kubermatic" --scopes="/subscriptions/********-****-****-****-************"
```

The output should look like the following one:

```text
{
  "appId": "********-****-****-****-************",
  "displayName": "azure-cli-2018-11-25-08-01-39",
  "name": "http://azure-cli-2018-11-25-08-01-39",
  "password": "********-****-****-****-************",
  "tenant": "********-****-****-****-************"
}
```

Enter provider credentials using the values from step "Prepare Azure Environment" into KKP Dashboard:

  - `Client ID`: Take the value of `appId`
  - `Client Secret`: Take the value of `password`
  - `Tenant ID`: your tenant ID
  - `Subscription ID`: your subscription ID

### Resources cleanup
During the machines cleanup, if KKP's Machine-Controller failed to delete the Cloud Provider instance and the user deleted
that instance manually, Machine-Controller won't be able to delete any referenced resources to that machine, such as Public
IPs, Disks and NICs. In that case, the user should cleanup those resources manually due to the fact that, Azure won't cleanup
any attached resources to the deleted instance.
