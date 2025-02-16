---
title: Identity and access management for SAP on Azure
description: Learn more about design considerations and recommendations that relate to identity and access management in an SAP deployment on Microsoft Azure.
author: pankajmeshramCSA
ms.author: pameshra
ms.date: 06/30/2023
ms.topic: conceptual
ms.subservice: caf-scenario-sap
ms.custom: think-tank, e2e-sap, UpdateFrequency2
---

<!-- docutune:casing LMS -->

# Identity and access management for SAP

This article builds on several considerations and recommendations defined in the article [Azure landing zone design area for identity and access management](../../ready/landing-zone/design-area/identity-access.md). This article describes the identity and access-management recommendations for deploying an SAP platform on Microsoft Azure. SAP is a mission-critical platform, so you should include the Azure landing zone design area guidance in your design.

## Design considerations

- Review the required Azure administration and management activities for your team. Consider your SAP on Azure landscape. Determine the best possible distribution of responsibilities within your organization.

- Determine the Azure resource administration boundaries versus the SAP Basis administration boundaries between the infrastructure and SAP Basis teams. Consider providing the SAP Basis team with elevated Azure resource administration access in an SAP non-production environment. For example, give them a [Virtual Machine Contributor](/azure/role-based-access-control/built-in-roles#virtual-machine-contributor) role. You can also give them partially elevated administration access, like partial Virtual Machine Contributor in a production environment. Both options achieve a good balance between the separation of duties and operational efficiency.

- For central IT and SAP Basis teams, consider using Privileged Identity Management (PIM) and multifactor authentication to access SAP Virtual Machine resources from the Azure portal and the underlying infrastructure.

Here are common administration and management activities of SAP on Azure:

| Azure resource | Azure resource provider | Activities |
|---|---|---|
| Virtual machines | Microsoft.Compute/virtualMachines | Start, stop, restart, deallocate, deploy, redeploy, change, resize, extensions, availability sets, proximity placement groups |
| Virtual machines | Microsoft.Compute/disks | Read and write to disk |
| Storage | Microsoft.Storage | Read, change on storage accounts (for example, boot diagnostics) |
| Storage | Microsoft.NetApp | Read, change on NetApp capacity pools and volumes |
| Storage | Microsoft.NetApp | ANF snapshots |
| Storage | Microsoft.NetApp | ANF cross-region replication |
| Networking | Microsoft.Network/networkInterfaces | Read, create, and change network interfaces |
| Networking | Microsoft.Network/loadBalancers | Read, create, and change load balancers |
| Networking | Microsoft.Network/networkSecurityGroups | Read NSG |
| Networking | Microsoft.Network/azureFirewalls | Read firewall |

- If you're using SAP Business Technology Platform (BTP) services, consider using principal propagation to forward an identity from the SAP BTP application to your SAP landscape by using SAP Cloud Connector.

- Consider Azure Active Directory (Azure AD) Provisioning service to automatically provision and deprovision users and groups to [SAP Analytics Cloud](/azure/active-directory/saas-apps/sap-analytics-cloud-provisioning-tutorial) and [SAP Identity Authentication](/azure/active-directory/saas-apps/sap-cloud-platform-identity-authentication-provisioning-tutorial).

- Consider that a migration to Azure might be an opportunity to review and realign identity and access management processes. Review the processes in your SAP landscape and the processes at your enterprise level:
  - Review the SAP dormant user lockout policies.
  - Review the SAP user password policy and align it with Azure AD.
  - Review the leavers, movers, and starters (LMS) procedures and align them with Azure AD. If you're using SAP Human Capital Management (HCM), SAP HCM likely drives the LMS process.

- Consider provisioning users from [SuccessFactors Employee Central](/azure/active-directory/saas-apps/sap-successfactors-inbound-provisioning-cloud-only-tutorial) into Azure AD, with optional write-back of email address to SuccessFactors.

- Secure Network File System (NFS) communication between Azure NetApp Files and Azure Virtual Machines with [NFS client encryption using Kerberos](/azure/azure-netapp-files/configure-kerberos-encryption). Azure NetApp Files supports Active Directory Domain Services (AD DS) and Azure Active Directory Domain Services (Azure AD DS) for Azure AD connections. Consider the [performance effect of Kerberos on NFS v4.1](/azure/azure-netapp-files/configure-kerberos-encryption#kerberos_performance).

- SAP Identity Management (IDM) integrates with Azure AD by using SAP cloud identity provisioning as a proxy service. Consider Azure AD as a central data source for users using SAP IDM. Secure the Network File System (NFS) communication between Azure NetApp Files and Azure Virtual Machines with NFS client encryption using Kerberos. Azure NetApp Files require either AD DS or Azure AD DS connection for Kerberos ticketing. Consider the performance effect of Kerberos on NFS v4.1.

- Secure Remote Function Call (RFC) connections between SAP systems with secure network communications (SNC) by using appropriate protection levels, like quality of protection (QoP). SNC protection generates some performance overhead. To protect RFC communication between application servers of the same SAP system, SAP recommends using network security instead of SNC. The following Azure services support SNC-protected RFC connections to an SAP target system: Providers of Azure Monitor for SAP Solutions, the self-hosted integration runtime in Azure Data Factory, and the on-premises data gateway in case of Power BI, Power Apps, Power Automate, Azure Analysis Services, and Azure Logic Apps. SNC is required to configure single sign-on (SSO) in these cases.

## Design recommendations

- Implement SSO by using Windows AD, Azure AD, or AD FS, depending on the access type, so that the end users can connect to SAP applications without a user ID and password once the central identity provider successfully authenticates them.
  - Implement SSO to SAP SaaS applications like [SAP Analytics Cloud](/azure/active-directory/saas-apps/sapboc-tutorial), [SAP Cloud Platform](/azure/active-directory/saas-apps/sap-hana-cloud-platform-tutorial), [Business by design](/azure/active-directory/saas-apps/sapbusinessbydesign-tutorial), [SAP Qualtrics](/azure/active-directory/saas-apps/qualtrics-tutorial) and [SAP C4C](/azure/active-directory/saas-apps/sap-customer-cloud-tutorial) with Azure AD using SAML.
  - Implement SSO to [SAP NetWeaver](/azure/active-directory/saas-apps/sap-netweaver-tutorial)-based web applications like [SAP Fiori](/azure/active-directory/saas-apps/sap-fiori-tutorial) and SAP Web GUI by using SAML.
  - You can implement SSO to SAP GUI by using SAP NetWeaver SSO or a partner solution.
  - For SSO for SAP GUI and web browser access, implement SNC – Kerberos/SPNEGO (simple and protected GSSAPI negotiation mechanism) due to its ease of configuration and maintenance. For SSO with X.509 client certificates, consider the SAP Secure Login Server, which is a component of the SAP SSO solution.
  - Implement [SSO by using OAuth for SAP NetWeaver](/azure/active-directory/saas-apps/sap-netweaver-tutorial#configure-sap-netweaver-for-oauth) to allow third-party or custom applications to access SAP NetWeaver OData services.
  - Implement [SSO to SAP HANA](/azure/active-directory/saas-apps/saphana-tutorial)

- Consider Azure AD an identity provider for SAP systems hosted on RISE. For more information, see [Integrating the Service with Azure AD](https://help.sap.com/docs/identity-authentication/identity-authentication/integrating-service-with-microsoft-azure-ad).
- For applications that access SAP, you might want to use [principal propagation to establish SSO](https://github.com/azuredevcollege/SAP/blob/master/sap-oauth-saml-flow/README.md).

- If you're using SAP BTP services or SaaS solutions that require SAP Identity Authentication Service (IAS), [consider implementing SSO between SAP Cloud Identity Authentication Services and Azure AD](/azure/active-directory/saas-apps/sap-hana-cloud-platform-identity-authentication-tutorial) to access those SAP services. This integration lets SAP IAS act as a proxy identity provider and forwards authentication requests to Azure AD as the central user store and identity provider.

- If you're using SAP SuccessFactors, consider using the Azure AD [automated user provisioning](/azure/active-directory/saas-apps/sap-successfactors-inbound-provisioning-cloud-only-tutorial). With this integration, as you add new employees to SAP SuccessFactors, you can automatically create their user accounts in Azure AD. Optionally, you can create user accounts in Microsoft 365 or other SaaS applications that are supported by Azure AD. Use write-back of the email address to SAP SuccessFactors.
