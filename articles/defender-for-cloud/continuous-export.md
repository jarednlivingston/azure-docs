---
title: Continuous export can send Microsoft Defender for Cloud's alerts and recommendations to Log Analytics workspaces or Azure Event Hubs
description: Learn how to configure continuous export of security alerts and recommendations to Log Analytics workspaces or Azure Event Hubs
ms.topic: how-to
ms.author: benmansheim
author: bmansheim
ms.date: 12/09/2021

---
# Continuously export Microsoft Defender for Cloud data

Microsoft Defender for Cloud generates detailed security alerts and recommendations. You can view them in the portal or through programmatic tools. You might also need to export some or all of this information for tracking with other monitoring tools in your environment. 

You fully customize *what* will be exported, and *where* it will go with **continuous export**. For example, you can configure it so that:

- All high severity alerts are sent to an Azure Event Hub
- All medium or higher severity findings from vulnerability assessment scans of your SQL servers are sent to a specific Log Analytics workspace
- Specific recommendations are delivered to an Event Hub or Log Analytics workspace whenever they're generated 
- The secure score for a subscription is sent to a Log Analytics workspace whenever the score for a control changes by 0.01 or more 

Even though the feature is called *continuous*, there's also an option to export weekly snapshots.

This article describes how to configure continuous export to Log Analytics workspaces or Azure Event Hubs.

> [!NOTE]
> If you need to integrate Defender for Cloud with a SIEM, see [Stream alerts to a SIEM, SOAR, or IT Service Management solution](export-to-siem.md).

> [!TIP]
> Defender for Cloud also offers the option to perform a one-time, manual export to CSV. Learn more in [Manual one-time export of alerts and recommendations](#manual-one-time-export-of-alerts-and-recommendations).


## Availability

|Aspect|Details|
|----|:----|
|Release state:|General availability (GA)|
|Pricing:|Free|
|Required roles and permissions:|<ul><li>**Security admin** or **Owner** on the resource group</li><li>Write permissions for the target resource.</li><li>If you're using the Azure Policy 'DeployIfNotExist' policies described below you'll also need permissions for assigning policies</li><li>To export data to Event Hub, you'll need Write permission on the Event Hub Policy.</li><li>To export to a Log Analytics workspace:<ul><li>if it **has the SecurityCenterFree solution**, you'll need a minimum of read permissions for the workspace solution: `Microsoft.OperationsManagement/solutions/read`</li><li>if it **doesn't have the SecurityCenterFree solution**, you'll need write permissions for the workspace solution: `Microsoft.OperationsManagement/solutions/action`</li><li>Learn more about [Azure Monitor and Log Analytics workspace solutions](../azure-monitor/insights/solutions.md)</li></ul></li></ul>|
|Clouds:|:::image type="icon" source="./media/icons/yes-icon.png"::: Commercial clouds<br>:::image type="icon" source="./media/icons/yes-icon.png"::: National (Azure Government, Azure China 21Vianet)| 



## What data types can be exported?

Continuous export can export the following data types whenever they change:

- Security alerts.
- Security recommendations.
- Security findings. These can be thought of as 'sub' recommendations and belong to a 'parent' recommendation. For example:
    - The recommendations [System updates should be installed on your machines (powered by Update Center)](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/e1145ab1-eb4f-43d8-911b-36ddf771d13f) and [System updates should be installed on your machines](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/4ab6e3c5-74dd-8b35-9ab9-f61b30875b27) each has one 'sub' recommendation per outstanding system update.
    - The recommendation [Machines should have vulnerability findings resolved](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/1195afff-c881-495e-9bc5-1486211ae03f) has a 'sub' recommendation for every vulnerability identified by the vulnerability scanner.
    > [!NOTE]
    > If you’re configuring a continuous export with the REST API, always include the parent with the findings. 
- Secure score per subscription or per control.
- Regulatory compliance data.


## Set up a continuous export 

You can configure continuous export from the Microsoft Defender for Cloud pages in Azure portal, via the REST API, or at scale using the supplied Azure Policy templates. Select the appropriate tab below for details of each.

### [**Use the Azure portal**](#tab/azure-portal)

### Configure continuous export from the Defender for Cloud pages in Azure portal

The steps below are necessary whether you're setting up a continuous export to Log Analytics workspace or Azure Event Hubs.

1. From Defender for Cloud's menu, open **Environment settings**.

1. Select the specific subscription for which you want to configure the data export.

1. From the sidebar of the settings page for that subscription, select **Continuous Export**.

    :::image type="content" source="./media/continuous-export/continuous-export-options-page.png" alt-text="Export options in Microsoft Defender for Cloud.":::

    Here you see the export options. There's a tab for each available export target. 

1. Select the data type you'd like to export and choose from the filters on each type (for example, export only high severity alerts).

1. Select the appropriate export frequency:
    - **Streaming** – assessments will be sent when a resource’s health state is updated (if no updates occur, no data will be sent).
    - **Snapshots** – a snapshot of the current state of the selected data types will be sent once a week per subscription. To identify snapshot data, look for the field ``IsSnapshot``.

1. Optionally, if your selection includes one of these recommendations, you can include the vulnerability assessment findings together with them:
    - [SQL databases should have vulnerability findings resolved](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/82e20e14-edc5-4373-bfc4-f13121257c37)
    - [SQL servers on machines should have vulnerability findings resolved](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/f97aa83c-9b63-4f9a-99f6-b22c4398f936)
    - [Container registry images should have vulnerability findings resolved (powered by Qualys)](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/dbd0cb49-b563-45e7-9724-889e799fa648)
    - [Machines should have vulnerability findings resolved](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/1195afff-c881-495e-9bc5-1486211ae03f)
    - [System updates should be installed on your machines](https://portal.azure.com/#blade/Microsoft_Azure_Security/RecommendationsBlade/assessmentKey/4ab6e3c5-74dd-8b35-9ab9-f61b30875b27)

    To include the findings with these recommendations, enable the **include security findings** option.

    :::image type="content" source="./media/continuous-export/include-security-findings-toggle.png" alt-text="Include security findings toggle in continuous export configuration." :::

1. From the "Export target" area, choose where you'd like the data saved. Data can be saved in a target on a different subscription (for example on a Central Event Hub instance or a central Log Analytics workspace).
1. Select **Save**.

### [**Use the REST API**](#tab/rest-api)

### Configure continuous export using the REST API

Continuous export can be configured and managed via the Microsoft Defender for Cloud [automations API](/rest/api/securitycenter/automations). Use this API to create or update rules for exporting to any of the following possible destinations:

- Azure Event Hub
- Log Analytics workspace
- Azure Logic Apps 

The API provides additional functionality not available from the Azure portal, for example:

* **Greater volume** - You can create multiple export configurations on a single subscription with the API. The **Continuous Export** page in Defender for Cloud's portal UI supports only one export configuration per subscription.

* **Additional features** - The API offers additional parameters that aren't shown in the UI. For example, you can add tags to your automation resource as well as define your export based on a wider set of alert and recommendation properties than those offered in the **Continuous Export** page in Defender for Cloud's portal UI.

* **More focused scope** - The API provides a more granular level for the scope of your export configurations. When defining an export with the API, you can do so at the resource group level. If you're using the **Continuous Export** page in Defender for Cloud's portal UI, you have to define it at the subscription level.

    > [!TIP]
    > If you've set up multiple export configurations using the API, or if you've used API-only parameters, those extra features will not be displayed in the Defender for Cloud UI. Instead, there'll be a banner informing you that other configurations exist.

Learn more about the automations API in the [REST API documentation](/rest/api/securitycenter/automations).

### [**Deploy at scale with Azure Policy**](#tab/azure-policy)

### Configure continuous export at scale using the supplied policies

Automating your organization's monitoring and incident response processes can greatly improve the time it takes to investigate and mitigate security incidents.

To deploy your continuous export configurations across your organization, use the supplied Azure Policy 'DeployIfNotExist' policies described below to create and configure continuous export procedures.

**To implement these policies**

1. From the table below, select the policy you want to apply:

    |Goal  |Policy  |Policy ID  |
    |---------|---------|---------|
    |Continuous export to Event Hub|[Deploy export to Event Hub for Microsoft Defender for Cloud alerts and recommendations](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fcdfcce10-4578-4ecd-9703-530938e4abcb)|cdfcce10-4578-4ecd-9703-530938e4abcb|
    |Continuous export to Log Analytics workspace|[Deploy export to Log Analytics workspace for Microsoft Defender for Cloud alerts and recommendations](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fffb6f416-7bd2-4488-8828-56585fef2be9)|ffb6f416-7bd2-4488-8828-56585fef2be9|


    > [!TIP]
    > You can also find these by searching Azure Policy:
    > 1. Open Azure Policy.
    > :::image type="content" source="./media/continuous-export/opening-azure-policy.png" alt-text="Accessing Azure Policy.":::
    > 2. From the Azure Policy menu, select **Definitions** and search for them by name. 

1. From the relevant Azure Policy page, select **Assign**.
    :::image type="content" source="./media/continuous-export/export-policy-assign.png" alt-text="Assigning the Azure Policy.":::

1. Open each tab and set the parameters as desired:
    1. In the **Basics** tab, set the scope for the policy. To use centralized management, assign the policy to the Management Group containing the subscriptions that will use continuous export configuration. 
    1. In the **Parameters** tab, set the resource group and data type details. 
        > [!TIP]
        > Each parameter has a tooltip explaining the options available to you.
        >
        > Azure Policy's parameters tab (1) provides access to similar configuration options as Defender for Cloud's continuous export page (2).
        > :::image type="content" source="./media/continuous-export/azure-policy-next-to-continuous-export.png" alt-text="Comparing the parameters in continuous export with Azure Policy." lightbox="./media/continuous-export/azure-policy-next-to-continuous-export.png":::
    1. Optionally, to apply this assignment to existing subscriptions, open the **Remediation** tab and select the option to create a remediation task.
1. Review the summary page and select **Create**.

--- 

## Information about exporting to a Log Analytics workspace

If you want to analyze Microsoft Defender for Cloud data inside a Log Analytics workspace or use Azure alerts together with Defender for Cloud alerts, set up continuous export to your Log Analytics workspace.

### Log Analytics tables and schemas

Security alerts and recommendations are stored in the *SecurityAlert* and *SecurityRecommendation* tables respectively. 

The name of the Log Analytics solution containing these tables depends on whether you have enabled the enhanced security features: Security ('Security and Audit') or SecurityCenterFree. 

> [!TIP]
> To see the data on the destination workspace, you must enable one of these solutions **Security and Audit** or **SecurityCenterFree**.

![The *SecurityAlert* table in Log Analytics.](./media/continuous-export/log-analytics-securityalert-solution.png)

To view the event schemas of the exported data types, visit the [Log Analytics table schemas](https://aka.ms/ASCAutomationSchemas).


##  View exported alerts and recommendations in Azure Monitor

You might also choose to view exported Security Alerts and/or recommendations in [Azure Monitor](../azure-monitor/alerts/alerts-overview.md). 

Azure Monitor provides a unified alerting experience for a variety of Azure alerts including Diagnostic Log, Metric alerts, and custom alerts based on Log Analytics workspace queries.

To view alerts and recommendations from Defender for Cloud in Azure Monitor, configure an Alert rule based on Log Analytics queries (Log Alert):

1. From Azure Monitor's **Alerts** page, select **New alert rule**.

    ![Azure Monitor's alerts page.](./media/continuous-export/azure-monitor-alerts.png)

1. In the create rule page, configure your new rule (in the same way you'd configure a [log alert rule in Azure Monitor](../azure-monitor/alerts/alerts-unified-log.md)):

    * For **Resource**, select the Log Analytics workspace to which you exported security alerts and recommendations.

    * For **Condition**, select **Custom log search**. In the page that appears, configure the query, lookback period, and frequency period. In the search query, you can type *SecurityAlert* or *SecurityRecommendation* to query the data types that Defender for Cloud continuously exports to as you enable the Continuous export to Log Analytics feature. 
    
    * Optionally, configure the [Action Group](../azure-monitor/alerts/action-groups.md) that you'd like to trigger. Action groups can trigger email sending, ITSM tickets, WebHooks, and more.
    ![Azure Monitor alert rule.](./media/continuous-export/azure-monitor-alert-rule.png)

You'll now see new Microsoft Defender for Cloud alerts or recommendations (depending on your configured continuous export rules and the condition you defined in your Azure Monitor alert rule) in Azure Monitor alerts, with automatic triggering of an action group (if provided).

## Manual one-time export of alerts and recommendations

To download a CSV report for alerts or recommendations, open the **Security alerts** or **Recommendations** page and select the **Download CSV report** button.

> [!TIP]
> Due to Azure Resource Graph limitations, the reports are limited to a file size of 13K rows. If you're seeing errors related to too much data being exported, try limiting the output by selecting a smaller set of subscriptions to be exported.

:::image type="content" source="./media/continuous-export/download-alerts-csv.png" alt-text="Download alerts data as a CSV file." lightbox="./media/continuous-export/download-alerts-csv.png":::

> [!NOTE]
> These reports contain alerts and recommendations for resources from the currently selected subscriptions.


## FAQ - Continuous export

### What are the costs involved in exporting data?

There is no cost for enabling a continuous export. Costs might be incurred for ingestion and retention of data in your Log Analytics workspace, depending on your configuration there. 

Learn more about [Log Analytics workspace pricing](https://azure.microsoft.com/pricing/details/monitor/).

Learn more about [Azure Event Hub pricing](https://azure.microsoft.com/pricing/details/event-hubs/).


### Does the export include data about the current state of all resources?

No. Continuous export is built for streaming of **events**:

- **Alerts** received before you enabled export won't be exported.
- **Recommendations** are sent whenever a resource's compliance state changes. For example, when a resource turns from healthy to unhealthy. Therefore, as with alerts, recommendations for resources that haven't changed state since you enabled export won't be exported.
- **Secure score** per security control or subscription is sent when a security control's score changes by 0.01 or more. 
- **Regulatory compliance status** is sent when the status of the resource's compliance changes.



### Why are recommendations sent at different intervals?

Different recommendations have different compliance evaluation intervals, which can vary from a few minutes to every few days. Consequently, recommendations will differ in the amount of time it takes for them to appear in your exports.

### Does continuous export support any business continuity or disaster recovery (BCDR) scenarios?

When preparing your environment for BCDR scenarios, where the target resource is experiencing an outage or other disaster, it's the organization's responsibility to prevent data loss by establishing backups according to the guidelines from Azure Event Hubs, Log Analytics workspace, and Logic App.

Learn more in [Azure Event Hubs - Geo-disaster recovery](../event-hubs/event-hubs-geo-dr.md).


### Is continuous export available for free?

Yes! Note that many alerts are only provided when you've enabled advanced protections. A good way to preview the alerts you'll get in your exported data is to see the alerts shown in Defender for Cloud's pages in the Azure portal.



## Next steps

In this article, you learned how to configure continuous exports of your recommendations and alerts. You also learned how to download your alerts data as a CSV file. 

For related material, see the following documentation: 

- Learn more about [workflow automation templates](https://github.com/Azure/Azure-Security-Center/tree/master/Workflow%20automation).
- [Azure Event Hubs documentation](../event-hubs/index.yml)
- [Microsoft Sentinel documentation](../sentinel/index.yml)
- [Azure Monitor documentation](../azure-monitor/index.yml)
- [Export data types schemas](https://aka.ms/ASCAutomationSchemas)
