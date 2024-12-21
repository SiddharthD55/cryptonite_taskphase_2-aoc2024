In this walkthrough, I’ll take you through my journey of learning about Azure, exploring its services, and delving into an assumed breach scenario that highlights how an attacker could exploit an Azure tenant's configuration.
Let’s begin with a brief introduction to Azure before diving into the Glitch’s assumed breach path and unraveling the critical security insights from the scenario.

## Introduction to Azure  

Azure is a **Cloud Service Provider (CSP)** that offers scalable computing resources on demand. CSPs like Azure (and competitors like AWS or Google Cloud) are game changers for managing infrastructure. Rather than relying on costly and time-consuming on-premises setups, Azure provides the flexibility to dynamically scale resources up or down and charges only for what’s used.

When I considered McSkidy’s use case, the benefits of Azure became clear. McSkidy started out managing her infrastructure on-premises for Wareville’s cybersecurity needs, handling resources like web applications for scheduling appointments, machines for forensic analysis, and evidence storage. As her workload grew, so did the complexity of scaling infrastructure during peak demands. Azure’s scalability and extensive suite of services were the perfect solution.  

Beyond cost optimization, Azure also provides **200+ cloud services**, making it a versatile platform for managing identity, securing data, and developing future-ready applications. Two key services stood out during this exploration:

### Azure Key Vault  

Azure Key Vault is designed for securely storing sensitive information like API keys, certificates, passwords, and cryptographic keys. Vault owners manage access, configure auditing, and define permissions for vault consumers, ensuring sensitive data stays protected. McSkidy used this service to store secrets critical to Wareville's operations.  

### Microsoft Entra ID  

Formerly known as Azure Active Directory, Microsoft Entra ID provides **Identity and Access Management (IAM)** capabilities. It helps manage user access to resources. For example, Wareville's town members had Entra ID accounts, with permissions configured by McSkidy to securely update or access their secrets.


## Assumed Breach Scenario  

To simulate an attack and evaluate the risks, I conducted an **Assumed Breach test** within the Wareville Azure tenant. This type of test assumes an attacker has already gained initial access to the network, allowing me to analyze how far they could escalate privileges and compromise resources.  

I started with valid credentials provided for the test environment. Using Azure Cloud Shell, I authenticated to the Azure portal and began my journey down the attack path.

## Connecting to the Azure Environment  

Azure Cloud Shell provides a browser-based CLI for managing Azure resources. It supports both **Bash** and **PowerShell**, making it an efficient tool for interacting with the tenant. I launched the Cloud Shell, selected Bash, and confirmed authentication using the `az ad signed-in-user show` command. This command displayed details about the authenticated user, confirming that the credentials worked.

## Enumeration: Users and Groups  

The first step in understanding the attack surface was enumerating users and groups. I used the Azure CLI command:  
```bash
az ad user list
```  
This command listed all users in the tenant. Since the list was overwhelming, I applied a filter to target users with the prefix `wvusr-`, which McSkidy indicated were more interesting:  
```bash
az ad user list --filter "startsWith('wvusr-', displayName)"
```

Among the results, one account stood out: **wvusr-backupware**. This account had unusual attributes, including a stored password in one of its fields. Additionally, I enumerated groups using the command:  
```bash
az ad group list
```

One group, **Secret Recovery Group**, had an intriguing description: _“Group for recovering Wareville’s secrets.”_ I listed the members of this group:  
```bash
az ad group member list --group "Secret Recovery Group"
```

The output confirmed that the `wvusr-backupware` account was a member of this sensitive group.

## Escalation via Azure Role Assignments  

Next, I investigated role assignments to determine the permissions of the Secret Recovery Group. Using the command:  
```bash
az role assignment list --assignee <GROUP_ID> --all
```

The results revealed that the group had two roles:  
- **Key Vault Secrets User**: Grants access to secret contents.  
- **Key Vault Reader**: Grants access to metadata but not the secret contents.  

Both roles were scoped to a key vault named `warevillesecrets`. These permissions allowed the group to read sensitive data stored in the vault.  

## Accessing the Azure Key Vault  

With this knowledge, I confirmed whether the `wvusr-backupware` account could exploit these permissions. First, I listed all accessible key vaults:  
```bash
az keyvault list
```

The `warevillesecrets` vault appeared in the output. Next, I listed the secrets stored in this vault:  
```bash
az keyvault secret list --vault-name warevillesecrets
```

Finally, I retrieved the contents of a specific secret:  
```bash
az keyvault secret show --vault-name warevillesecrets --name <SECRET_NAME>
```

This command revealed the secret value, confirming that the `Key Vault Secrets User` role allowed full access to sensitive data.

## Lessons Learned  

This demonstrated the importance of properly configuring Azure Role Assignments and securing sensitive accounts. Misconfigurations like assigning excessive privileges to a group can create significant risks. 

1. **Limit Privileges**: Avoid granting roles like `Key Vault Secrets User` unless absolutely necessary.  
2. **Audit Roles and Assignments**: Regularly review role assignments to ensure compliance with the principle of least privilege.  
3. **Secure Sensitive Accounts**: Protect accounts like `wvusr-backupware` by enforcing multi-factor authentication (MFA) and auditing their use.  

By identifying these issues and addressing them, McSkidy can better secure Wareville’s Azure tenant and mitigate the risk of future breaches.

Well this assumed breach scenario was an eye-opening experience in Azure security. It highlighted the critical role of proper configuration and privilege management in safeguarding cloud environments. 

**Answers**

*What is the password for backupware that was leaked?* **R3c0v3r_s3cr3ts!**
 
*What is the group ID of the Secret Recovery Group?* **7d96660a-02e1-4112-9515-1762d0cb66b7**
 
*What is the name of the vault secret?* **aoc2024**
 
*What are the contents of the secret stored in the vault?* **WhereIsMyMind1999**
