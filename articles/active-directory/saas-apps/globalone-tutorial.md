---
title: 'Tutorial: Azure AD SSO integration with EY GlobalOne'
description: Learn how to configure single sign-on between Azure Active Directory and EY GlobalOne.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/22/2021
ms.author: jeedes
---

# Tutorial: Azure AD SSO integration with EY GlobalOne

In this tutorial, you'll learn how to integrate EY GlobalOne with Azure Active Directory (Azure AD). When you integrate EY GlobalOne with Azure AD, you can:

* Control in Azure AD who has access to EY GlobalOne.
* Enable your users to be automatically signed-in to EY GlobalOne with their Azure AD accounts.
* Manage your accounts in one central location - the Azure portal.

## Prerequisites

To get started, you need the following items:

* An Azure AD subscription. If you don't have a subscription, you can get a [free account](https://azure.microsoft.com/free/).
* EY GlobalOne single sign-on (SSO) enabled subscription.

## Scenario description

In this tutorial, you configure and test Azure AD SSO in a test environment.
* EY GlobalOne supports **SP and IDP** initiated SSO. 
* EY GlobalOne supports **Just In Time** user provisioning.

## Add EY GlobalOne from the gallery

To configure the integration of EY GlobalOne into Azure AD, you need to add EY GlobalOne from the gallery to your list of managed SaaS apps.

1. Sign in to the Azure portal using either a work or school account, or a personal Microsoft account.
1. On the left navigation pane, select the **Azure Active Directory** service.
1. Navigate to **Enterprise Applications** and then select **All Applications**.
1. To add new application, select **New application**.
1. In the **Add from the gallery** section, type **EY GlobalOne** in the search box.
1. Select **EY GlobalOne** from results panel and then add the app. Wait a few seconds while the app is added to your tenant.

## Configure and test Azure AD SSO for EY GlobalOne

Configure and test Azure AD SSO with EY GlobalOne using a test user called **B. Simon**. For SSO to work, you need to establish a link relationship between an Azure AD user and the related user in EY GlobalOne.

To configure and test Azure AD SSO with EY GlobalOne, perform the following steps:

1. **[Configure Azure AD SSO](#configure-azure-ad-sso)** to enable your users to use this feature.
	1. **[Create an Azure AD test user](#create-an-azure-ad-test-user)** to test Azure AD single sign-on with B. Simon.
	1. **[Assign the Azure AD test user](#assign-the-azure-ad-test-user)** to enable B. Simon to use Azure AD single sign-on.
1. **[Configure EY GlobalOne SSO](#configure-ey-globalone-sso)** to configure the SSO settings on application side.
	1. **[Create EY GlobalOne test user](#create-ey-globalone-test-user)** to have a counterpart of B. Simon in EY GlobalOne that is linked to the Azure AD representation of user.
1. **[Test SSO](#test-sso)** to verify whether the configuration works.

## Configure Azure AD SSO

Follow these steps to enable Azure AD SSO in the Azure portal.

1. In the Azure portal, on the **EY GlobalOne** application integration page, find the **Manage** section and select **Single sign-on**.
1. On the **Select a Single sign-on method** page, select **SAML**.
1. On the **Set up Single Sign-On with SAML** page, click the pencil icon for **Basic SAML Configuration** to edit the settings.

   ![Edit Basic SAML Configuration](common/edit-urls.png)

1. On the **Basic SAML Configuration** section, the application is pre-configured and the necessary URLs are already pre-populated with Azure. The user needs to save the configuration by clicking the **Save** button.

1. EY GlobalOne application expects the SAML assertions in a specific format, which requires you to add custom attribute mappings to your SAML token attributes configuration. The following screenshot shows the list of default attributes. Click **Edit** icon to open User Attributes dialog.

	![Screenshot that shows the "User Attributes" section with the "Edit" icon selected.](common/edit-attribute.png)

1. In addition to above, EY GlobalOne application expects few more attributes to be passed back in SAML response. In the **User Claims** section on the **User Attributes** dialog, perform the following steps to add SAML token attribute as shown in the below table:

	| Name | Source Attribute|
	| ---------------| --------------- |
	| FirstName | user.givenname |
	| LastName | user.surname |
	| Email | user.mail |
	| Company | `<YOUR COMPANY NAME>` |

	a. Click **Add new claim** to open the **Manage user claims** dialog.

	![Screenshot that shows the "User claims" section with the "Add new claim" and "Save" actions highlighted.](common/new-save-attribute.png)

	![image](common/new-attribute-details.png)

	b. In the **Name** textbox, type the attribute name shown for that row.

	c. Leave the **Namespace** blank.

	d. Select Source as **Attribute**.

	e. From the **Source attribute** list, type the attribute value shown for that row.

	f. Click **Ok**

	g. Click **Save**.

1. On the **Set up Single Sign-On with SAML** page, in the **SAML Signing Certificate** section, find **Certificate (Raw)** and select **Download** to download the certificate and save it on your computer.

   ![The Certificate download link](common/certificateraw.png)

1. On the **Set up EY GlobalOne** section, copy the appropriate URL(s) based on your requirement.

   ![Copy configuration URLs](common/copy-configuration-urls.png)

### Create an Azure AD test user

In this section, you'll create a test user in the Azure portal called B. Simon.

1. From the left pane in the Azure portal, select **Azure Active Directory**, select **Users**, and then select **All users**.
1. Select **New user** at the top of the screen.
1. In the **User** properties, follow these steps:
   1. In the **Name** field, enter `B. Simon`.  
   1. In the **User name** field, enter the username@companydomain.extension. For example, `B. Simon@contoso.com`.
   1. Select the **Show password** check box, and then write down the value that's displayed in the **Password** box.
   1. Click **Create**.

### Assign the Azure AD test user

In this section, you'll enable B. Simon to use Azure single sign-on by granting access to EY GlobalOne.

1. In the Azure portal, select **Enterprise Applications**, and then select **All applications**.
1. In the applications list, select **EY GlobalOne**.
1. In the app's overview page, find the **Manage** section and select **Users and groups**.
1. Select **Add user**, then select **Users and groups** in the **Add Assignment** dialog.
1. In the **Users and groups** dialog, select **B. Simon** from the Users list, then click the **Select** button at the bottom of the screen.
1. If you're expecting any role value in the SAML assertion, in the **Select Role** dialog, select the appropriate role for the user from the list and then click the **Select** button at the bottom of the screen.
1. In the **Add Assignment** dialog, click the **Assign** button.

## Configure EY GlobalOne SSO

To configure single sign-on on **EY GlobalOne** side, you need to send the downloaded **Certificate (Raw)** and appropriate copied URLs from Azure portal to [EY GlobalOne support team](mailto:globalone.support@ey.com). They set this setting to have the SAML SSO connection set properly on both sides.

### Create EY GlobalOne test user

In this section, a user called Britta Simon is created in EY GlobalOne. EY GlobalOne supports just-in-time user provisioning, which is enabled by default. There is no action item for you in this section. If a user doesn't already exist in EY GlobalOne, a new one is created after authentication.

## Test SSO

In this section, you test your Azure AD single sign-on configuration with following options. 

#### SP initiated:

* Click on **Test this application** in Azure portal. This will redirect to EY GlobalOne Sign on URL where you can initiate the login flow.  

* Go to EY GlobalOne Sign-on URL directly and initiate the login flow from there.

#### IDP initiated:

* Click on **Test this application** in Azure portal and you should be automatically signed in to the EY GlobalOne for which you set up the SSO. 

You can also use Microsoft My Apps to test the application in any mode. When you click the EY GlobalOne tile in the My Apps, if configured in SP mode you would be redirected to the application sign on page for initiating the login flow and if configured in IDP mode, you should be automatically signed in to the EY GlobalOne for which you set up the SSO. For more information about the My Apps, see [Introduction to the My Apps](../user-help/my-apps-portal-end-user-access.md).

## Next steps

Once you configure EY GlobalOne you can enforce session control, which protects exfiltration and infiltration of your organization’s sensitive data in real time. Session control extends from Conditional Access. [Learn how to enforce session control with Microsoft Defender for Cloud Apps](/cloud-app-security/proxy-deployment-aad).
