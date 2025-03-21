---
title: Add a connected organization in Azure AD entitlement management - Azure Active Directory
description: Learn how to allow people outside your organization to request access packages so that you can collaborate on projects.
services: active-directory
documentationCenter: ''
author: owinfreyatl
manager: amycolannino
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/11/2020
ms.author: owinfrey
ms.reviewer: mwahl
ms.collection: M365-identity-device-management


#Customer intent: As an administrator, I want to allow users in certain partner organizations to request access packages so that our organizations can collaborate on projects.

---

# Add a connected organization in Azure AD entitlement management

With Azure Active Directory (Azure AD) entitlement management, you can collaborate with people outside your organization. If you frequently collaborate with users in an external Azure AD directory or domain, you can add them as a connected organization. This article describes how to add a connected organization so that you can allow users outside your organization to request resources in your directory.

## What is a connected organization?

A connected organization is another organization that you have a relationship with.  In order for the users in that organization to be able to access your resources, such as your SharePoint Online sites or apps, you'll need a representation of that organization's users in that directory.  Because in most cases the users in that organization aren't already in your Azure AD directory, you can use entitlement management to bring them into your Azure AD directory as needed.  

There are three ways that entitlement management lets you specify the users that form a connected organization.  It could be

* users in another Azure AD directory (from any Microsoft cloud),
* users in another non-Azure AD directory that has been configured for direct federation, or
* users in another non-Azure AD directory, whose email addresses all have the same domain name in common.

In addition, you can have a connected organization for users with a Microsoft Account, such as from the domain *live.com*.

For example, suppose you work at Woodgrove Bank and you want to collaborate with two external organizations. These two organizations have different configurations:

- Graphic Design Institute uses Azure AD, and their users have a user principal name that ends with *graphicdesigninstitute.com*.
- Contoso does not yet use Azure AD. Contoso users have a user principal name that ends with *contoso.com*.

In this case, you can configure two connected organizations. You create one connected organization for Graphic Design Institute and one for Contoso. If you then add the two connected organizations to a policy, users from each organization with a user principal name that matches the policy can request access packages. Users with a user principal name that has a domain of contoso.com would match the Contoso-connected organization and would also be allowed to request packages. Users with a user principal name that has a domain of *graphicdesigninstitute.com* would match the Graphic Design Institute-connected organization and be allowed to submit requests. And, because Graphic Design Institute uses Azure AD, any users with a principal name that matches a [verified domain](../fundamentals/add-custom-domain.md#verify-your-custom-domain-name) that's added to their tenant, such as *graphicdesigninstitute.example*, would also be able to request access packages by using the same policy. If you have [email one-time passcode (OTP) authentication](../external-identities/one-time-passcode.md) turned on, that includes users from those domains that aren't yet part of Azure AD directories who'll authenticate using email OTP when accessing your resources.

![Connected organization example](./media/entitlement-management-organization/connected-organization-example.png)

How users from the Azure AD directory or domain authenticate depends on the authentication type. The authentication types for connected organizations are:

- Azure AD
- [Direct federation](../external-identities/direct-federation.md)
- [One-time passcode](../external-identities/one-time-passcode.md) (domain)

For a demonstration of how to add a connected organization, watch the following video:

>[!VIDEO https://www.microsoft.com/videoplayer/embed/RE4dskS]

## Add a connected organization

To add an external Azure AD directory or domain as a connected organization, follow the instructions in this section.

**Prerequisite role**: *Global administrator*, *Identity Governance administrator*,  or *User administrator*

1. In the Azure portal, select **Azure Active Directory**, and then select **Identity Governance**.

1. In the left pane, select **Connected organizations**, and then select **Add connected organization**.

    ![The "Add connected organization" button](./media/entitlement-management-organization/connected-organization.png)

1. Select the **Basics** tab, and then enter a display name and description for the organization.

    ![The "Add connected organization" Basics pane](./media/entitlement-management-organization/organization-basics.png)

1. The state will automatically be set to **Configured** when you create a new connected organization. For more information about state properties, see [State properties of connected organizations](#state-properties-of-connected-organizations)

1. Select the **Directory + domain** tab, and then select **Add directory + domain**.

    Then **Select directories + domains** pane opens.

1. In the search box, enter a domain name to search for the Azure AD directory or domain. You can also add domains that are not in Azure AD. Be sure to enter the entire domain name.

1. Confirm that the organization name(s) and authentication type(s) are correct. User sign in, prior to being able to access the MyAccess portal, depends on the authentication type for their organization.  If the authentication type for a connected organization is Azure AD, all users with an account in any verified domain of that Azure AD directory will sign into their directory, and then can request access to access packages that allow that connected organization. If the authentication type is One-time passcode, this allows users with email addresses from just that domain to visit the MyAccess portal. After they authenticate with the passcode, the user can make a request.

    ![The "Select directories + domains" pane](./media/entitlement-management-organization/organization-select-directories-domains.png)

    > [!NOTE]
    > Access from some domains could be blocked by the Azure AD business to business (B2B) allow or deny list. For more information, see [Allow or block invitations to B2B users from specific organizations](../external-identities/allow-deny-list.md).

1. Select **Add** to add the Azure AD directory or domain. **You can add multiple Azure AD directories and domains**.

1. After you've added the Azure AD directories or domains, select **Select**.

    The organization(s) appears in the list.

    ![The "Directory + domain" pane](./media/entitlement-management-organization/organization-directory-domain.png)

1. Select the **Sponsors** tab, and then add optional sponsors for this connected organization.

    Sponsors are internal or external users already in your directory that are the point of contact for the relationship with this connected organization. Internal sponsors are member users in your directory. External sponsors are guest users from the connected organization that were previously invited and are already in your directory. Sponsors can be utilized as approvers when users in this connected organization request access to this access package. For information about how to invite a guest user to your directory, see [Add Azure Active Directory B2B collaboration users in the Azure portal](../external-identities/add-users-administrator.md).

    When you select **Add/Remove**, a pane opens in which you can choose internal or external sponsors. The pane displays an unfiltered list of users and groups in your directory.

    ![The Sponsors pane](./media/entitlement-management-organization/organization-sponsors.png)

1. Select the **Review + create** tab, review your organization settings, and then select **Create**.

    ![The "Review + create" pane](./media/entitlement-management-organization/organization-review-create.png)

## Update a connected organization 

If the connected organization changes to a different domain, the organization's name changes, or you want to change the sponsors, you can update the connected organization by following the instructions in this section.

**Prerequisite role**: *Global administrator* or *User administrator*

1. In the Azure portal, select **Azure Active Directory**, and then select **Identity Governance**.

1. In the left pane, select **Connected organizations**, and then select the connected organization to open it.

1. In the connected organization's overview pane, select **Edit** to change the organization name, description, or state.  

1. In the **Directory + domain** pane, select **Update directory + domain** to change to a different directory or domain.

1. In the **Sponsors** pane, select **Add internal sponsors** or **Add external sponsors** to add a user as a sponsor. To remove a sponsor, select the sponsor and, in the right pane, select **Delete**.


## Delete a connected organization

If you no longer have a relationship with an external Azure AD directory or domain, you can delete the connected organization.

**Prerequisite role**: *Global administrator* or *User administrator*

1. In the Azure portal, select **Azure Active Directory**, and then select **Identity Governance**.

1. In the left pane, select **Connected organizations**, and then select the connected organization to open it.

1. In the connected organization's overview pane, select **Delete** to delete it.

    ![The connected organization Delete button](./media/entitlement-management-organization/organization-delete.png)

## Managing a connected organization programmatically

You can also create, list, update, and delete connected organizations using Microsoft Graph. A user in an appropriate role with an application that has the delegated `EntitlementManagement.ReadWrite.All` permission can call the API to manage [connectedOrganization](/graph/api/resources/connectedorganization) objects and set sponsors for them.

## State properties of connected organizations

There are two different types of state properties for connected organizations in Azure AD entitlement management currently, configured and proposed: 

- A configured connected organization is a fully functional connected organization that allows users within that organization access to access packages. When an admin creates a new connected organization in the Azure portal, it will be in the **configured** state by default since the administrator created and wants to use this connected organization. Additionally, when a connected org is created programmatically via the API, the default state should be **configured** unless set to another state explicitly. 

    Configured connected organizations will show up in the pickers for connected organizations and will be in scope for any policies that target “all” connected organizations.

- A proposed connected organization is a connected organization that has been automatically created, but hasn't had an administrator create or approve the organization. When a user signs up for an access package outside of a configured connected organization, any automatically created connected organizations will be in the **proposed** state since no administrator in the tenant set-up that partnership. 
    
    Proposed connected organizations are not in scope for the “all configured connected organizations” setting on any policies but can be used in policies only for policies targeting specific organizations. 

Only users from configured connected organizations can request access packages that are available to users from all configured organizations. Users from proposed connected organizations have an experience as if there is no connected organization for that domain; can only see and request access packages scoped to their specific organization or scoped to any user.

> [!NOTE]
> As part of rolling out this new feature, all connected organizations created before 09/09/20 were considered **configured**. If you had an access package that allowed users from any organization to sign up, you should review your list of connected organizations that were created before that date to ensure none are miscategorized as **configured**.  An admin can update the **State** property as appropriate. For guidance, see [Update a connected organization](#update-a-connected-organization).
> [!NOTE]
>  In some cases, a user might request an access package using their personal account having the same domain as the connected organization resulting in a new suggested connected organization. In this case, make sure the user is using their organization account instead and the portal will identify this user coming from the configured connected organization Azure AD tenant.


## Next steps

- [Govern access for external users](./entitlement-management-external-users.md)
- [Govern access for users not in your directory](entitlement-management-access-package-request-policy.md#for-users-not-in-your-directory)
