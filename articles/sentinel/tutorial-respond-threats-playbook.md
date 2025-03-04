---
title: Tutorial - Automate threat response in Microsoft Sentinel
description: Use this tutorial to help you use playbooks together with automation rules in Microsoft Sentinel to automate your incident response and remediate security threats.
author: yelevin
ms.author: yelevin
ms.topic: tutorial
ms.date: 05/09/2023
---

# Tutorial: Respond to threats by using playbooks with automation rules in Microsoft Sentinel

This tutorial shows you how to use playbooks together with automation rules to automate your incident response and remediate security threats detected by Microsoft Sentinel. When you complete this tutorial you will be able to:

> [!div class="checklist"]
>
> * Create an automation rule
> * Create a playbook
> * Add actions to a playbook
> * Attach a playbook to an automation rule or an analytics rule to automate threat response

> [!NOTE]
> This tutorial provides basic guidance for a top customer task: creating automation to triage incidents. For more information, see our **How-to** section, such as [Automate threat response with playbooks in Microsoft Sentinel](automate-responses-with-playbooks.md) and [Use triggers and actions in Microsoft Sentinel playbooks](playbook-triggers-actions.md).
>

## What are automation rules and playbooks?

[Automation rules](automate-incident-handling-with-automation-rules.md) help you triage incidents in Microsoft Sentinel. You can use them to automatically assign incidents to the right personnel, close noisy incidents or known [false positives](false-positives.md), change their severity, and add tags. They are also the mechanism by which you can run playbooks in response to incidents or alerts.

Playbooks are collections of procedures that can be run from Microsoft Sentinel in response to an **entire incident**, to an **individual alert**, or to a **specific entity**. A playbook can help automate and orchestrate your response, and can be set to run automatically when specific alerts are generated or when incidents are created or updated, by being attached to an automation rule. It can also be run manually on-demand on specific incidents, alerts, or entities.

Playbooks in Microsoft Sentinel are based on workflows built in [Azure Logic Apps](../logic-apps/logic-apps-overview.md), which means that you get all the power, customizability, and built-in templates of Logic Apps. Each playbook is created for the specific subscription to which it belongs, but the **Playbooks** display shows you all the playbooks available across any selected subscriptions.

> [!NOTE]
> Because playbooks make use of Azure Logic Apps, additional charges may apply. Visit the [Azure Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/) pricing page for more details.

For example, if you want to stop potentially compromised users from moving around your network and stealing information, you can create an automated, multifaceted response to incidents generated by rules that detect compromised users. You start by creating a playbook that takes the following actions:

1. When the playbook is called by an automation rule passing it an incident, the playbook opens a ticket in [ServiceNow](/connectors/service-now/) or any other IT ticketing system.

1. It sends a message to your security operations channel in [Microsoft Teams](/connectors/teams/) or [Slack](/connectors/slack/) to make sure your security analysts are aware of the incident.

1. It also sends all the information in the incident in an email message to your senior network admin and security admin. The email message will include **Block** and **Ignore** user option buttons.

1. The playbook waits until a response is received from the admins, then continues with its next steps.

1. If the admins choose **Block**, it sends a command to Azure AD to disable the user, and one to the firewall to block the IP address.

1. If the admins choose **Ignore**, the playbook closes the incident in Microsoft Sentinel, and the ticket in ServiceNow.

In order to trigger the playbook, you'll then create an automation rule that runs when these incidents are generated. That rule will take these steps:

1. The rule changes the incident status to **Active**.

1. It assigns the incident to the analyst tasked with managing this type of incident.

1. It adds the "compromised user" tag.

1. Finally, it calls the playbook you just created. ([Special permissions are required for this step](automate-responses-with-playbooks.md#incident-creation-automated-response).)

Playbooks can be run automatically in response to incidents, by creating automation rules that call the playbooks as actions, as in the example above. They can also be run automatically in response to alerts, by telling the analytics rule to automatically run one or more playbooks when the alert is generated. 

You can also choose to run a playbook manually on-demand, as a response to a selected alert.

Get a more complete and detailed introduction to automating threat response using [automation rules](automate-incident-handling-with-automation-rules.md) and [playbooks](automate-responses-with-playbooks.md) in Microsoft Sentinel.

## Create a playbook

Follow these steps to create a new playbook in Microsoft Sentinel:

:::image type="content" source="./media/tutorial-respond-threats-playbook/add-new-playbook.png" alt-text="Screenshot of the menu selection for adding a new playbook in the Automation screen." lightbox="media/tutorial-respond-threats-playbook/add-new-playbook.png":::

1. From the **Microsoft Sentinel** navigation menu, select **Automation**.

1. From the top menu, select **Create**.

1. The drop-down menu that appears under **Create** gives you four choices for creating playbooks:

    1. If you're creating a **Standard** playbook (the new kind - see [Logic app types](automate-responses-with-playbooks.md#logic-app-types)), select **Blank playbook** and then follow the steps in the **Logic Apps Standard** tab below.

    1. If you're creating a **Consumption** playbook (the original, classic kind), then, depending on which trigger you want to use, select either **Playbook with incident trigger**, **Playbook with alert trigger**, or **Playbook with entity trigger**. Then, continue following the steps in the **Logic Apps Consumption** tab below.

        For more about which trigger to use, see [**Use triggers and actions in Microsoft Sentinel playbooks**](playbook-triggers-actions.md).

# [Logic Apps Consumption](#tab/LAC)
### Prepare the playbook and Logic App

Regardless of which trigger you chose to create your playbook with in the previous step, the **Create playbook** wizard will appear.

   :::image type="content" source="./media/tutorial-respond-threats-playbook/create-playbook-basics.png" alt-text="Create a logic app":::

1. In the **Basics** tab:

    1. Select the **Subscription**, **Resource group**, and **Region** of your choosing from their respective drop-down lists. The chosen region is where your Logic App information will be stored.

    1. Enter a name for your playbook under **Playbook name**.

    1. If you want to monitor this playbook's activity for diagnostic purposes, mark the **Enable diagnostics logs in Log Analytics** check box, and choose your **Log Analytics workspace** from the drop-down list.

    1. If your playbooks need access to protected resources that are inside or connected to an Azure virtual network, [you may need to use an integration service environment (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md). If so, mark the **Associate with integration service environment** check box, and select the desired ISE from the drop-down list.

    1. Select **Next : Connections >**.

1. In the **Connections** tab:

    Ideally you should leave this section as is, configuring Logic Apps to connect to Microsoft Sentinel with managed identity. [Learn about this and other authentication alternatives](authenticate-playbooks-to-sentinel.md).

    Select **Next : Review and create >**.

1. In the **Review and create** tab:

    Review the configuration choices you have made, and select **Create and continue to designer**.

1. Your playbook will take a few minutes to be created and deployed, after which you will see the message "Your deployment is complete" and you will be taken to your new playbook's [Logic App Designer](../logic-apps/logic-apps-overview.md). The trigger you chose at the beginning will have automatically been added as the first step, and you can continue designing the workflow from there.

    :::image type="content" source="media/tutorial-respond-threats-playbook/logic-app-blank.png" alt-text="Screenshot of logic app designer screen with opening trigger." lightbox="media/tutorial-respond-threats-playbook/logic-app-blank.png":::

    If you chose the **Microsoft Sentinel entity (Preview)** trigger, select the type of entity you want this playbook to receive as an input.

    :::image type="content" source="media/tutorial-respond-threats-playbook/entity-trigger-types.png" alt-text="Screenshot of drop-down list of entity types to choose from to set playbook schema.":::

# [Logic Apps Standard](#tab/LAS)

### Prepare the Logic App and workflow

There are three steps to getting started creating a Logic Apps Standard playbook:

1. [Create a Logic App](#create-a-logic-app).
1. [Create a workflow](#create-a-workflow-playbook) (this is the actual playbook).
1. [Choose the trigger](#choose-the-trigger).

#### Create a Logic App

Since you selected **Blank playbook**, a new browser tab will open and take you to the **Create Logic App** wizard.

   :::image type="content" source="./media/tutorial-respond-threats-playbook/create-logic-app-basics.png" alt-text="Create a Standard logic app.":::

1. In the **Basics** tab:

    1. Select the **Subscription** and **Resource Group** of your choosing from their respective drop-down lists.

    1. Enter a name for your Logic App. For **Publish**, choose **Workflow**. Select the **Region** where you wish to deploy the logic app.

    1. For **Plan type**, choose **Standard**.

    1. Select **Next : Hosting >**.

1. In the **Hosting** tab:

    1. For **Storage type**, choose **Azure Storage**, and choose or create a **Storage account**.

    1. Choose a **Windows Plan**.

1. Select **Next : Monitoring >**.

1. In the **Monitoring** tab:

    1. If you want to enable performance monitoring in Azure Monitor for this application, leave the toggle on Yes. Otherwise, toggle it to No.
    
        > [!NOTE]
        > This monitoring is **not required for Microsoft Sentinel** and **will cost you extra**.

    1. If you want you can select **Next : Tags >**  to apply tags to this Logic App for resource categorization and billing purposes. Otherwise, select **Review + create**.

1. In the **Review + create** tab:

    Review the configuration choices you have made, and select **Create**.

1. Your playbook will take a few minutes to be created and deployed, during which you will see some deployment messages. At the end of the process you will be taken to the final deployment screen where you'll see the message "Your deployment is complete".

    Select **Go to resource**. You will be taken to the main page of your new Logic App.

    Unlike with classic Consumption playbooks, you're not done yet. Now you must create a workflow.

#### Create a workflow (playbook)

1. Select **Workflows** from the navigation menu of your Logic App page.

1. Select **+ Add** from the button bar at the top (it might take a few seconds for the button to be active).

1. The **New workflow** panel will appear. Enter a name for your workflow.

1. Under **State type**, select **Stateful**.

    > [!NOTE]
    > Microsoft Sentinel does not currently support the use of *Stateless* workflows as playbooks.

1. Select **Create**. Your workflow will be saved and will appear in the list of workflows in your Logic App. Select the workflow to proceed.

1. You'll enter your workflow's page. Here you can see all the information about your workflow, including a record of all the times it will have run. From the navigation menu, select **Designer**.

1. The Designer screen will open and you will immediately be prompted to add a trigger and continue designing the workflow. 

    :::image type="content" source="media/tutorial-respond-threats-playbook/logic-app-standard-designer.png" alt-text="Screenshot of Logic App Standard designer." lightbox="media/tutorial-respond-threats-playbook/logic-app-standard-designer.png":::

#### Choose the trigger

1. Select the **Azure** tab and enter "Sentinel" in the Search line.

1. In the **Triggers** tab below, you will see the three triggers offered by Microsoft Sentinel:
    - Microsoft Sentinel alert (preview)
    - Microsoft Sentinel entity (preview)
    - Microsoft Sentinel incident (preview)

   Select the trigger that matches the type of playbook you are creating.

    :::image type="content" source="./media/tutorial-respond-threats-playbook/sentinel-triggers.png" alt-text="Choose a trigger for your playbook":::

    If you chose the **Microsoft Sentinel entity (Preview)** trigger, select the type of entity you want this playbook to receive as an input.

    :::image type="content" source="media/tutorial-respond-threats-playbook/entity-trigger-types-standard.png" alt-text="Screenshot of drop-down list of entity types to choose from to set playbook schema.":::

> [!NOTE]
> When you choose a trigger, or any subsequent action, you will be asked to authenticate to whichever resource provider you are interacting with. In this case, the provider is Microsoft Sentinel. There are a few different approaches you can take to authentication. For details and instructions, see [**Authenticate playbooks to Microsoft Sentinel**](authenticate-playbooks-to-sentinel.md).

For more about which trigger to use, see [**Use triggers and actions in Microsoft Sentinel playbooks**](playbook-triggers-actions.md)


---

### Add actions

Now you can define what happens when you call the playbook. You can add actions, logical conditions, loops, or switch case conditions, all by selecting **New step**. This selection opens a new frame in the designer, where you can choose a system or an application to interact with or a condition to set. Enter the name of the system or application in the search bar at the top of the frame, and then choose from the available results.

In every one of these steps, clicking on any field displays a panel with two menus: **Dynamic content** and **Expression**. From the **Dynamic content** menu, you can add references to the attributes of the alert or incident that was passed to the playbook, including the values and attributes of all the [mapped entities](map-data-fields-to-entities.md) and [custom details](surface-custom-details-in-alerts.md) contained in the alert or incident. From the **Expression** menu, you can choose from a large library of functions to add additional logic to your steps.

This screenshot shows the actions and conditions you would add in creating the playbook described in the example at the beginning of this document. Learn more about [adding actions to your playbooks](playbook-triggers-actions.md).

   :::image type="content" source="./media/tutorial-respond-threats-playbook/logic-app.png" alt-text="Screenshot showing the Logic App designer with an incident trigger workflow.":::

See [**Use triggers and actions in Microsoft Sentinel playbooks**](playbook-triggers-actions.md) for details about actions you can add to playbooks for different purposes.

In particular, note this important information about [**playbooks based on the entity trigger in a non-incident context**](playbook-triggers-actions.md#entity-playbooks-with-no-incident-id).

## Automate threat responses

You've created your playbook and defined the trigger, set the conditions, and prescribed the actions that it will take and the outputs it will produce. Now you need to determine the criteria under which it will run and set up the automation mechanism that will run it when those criteria are met.

### Respond to incidents and alerts
<a name="respond-to-incidents"></a><a name="respond-to-alerts"></a> <!-- Anchor links included here for backward compatibility with, and redirection of, old headings -->

To use a playbook to respond automatically to an **entire incident** or to an **individual alert**, create an [automation rule](automate-incident-handling-with-automation-rules.md) that will run when the incident is created or updated, or when the alert is generated. This automation rule will include a step that calls the playbook you want to use.

To create an automation rule:

1. From the **Automation** blade in the Microsoft Sentinel navigation menu, select **Create** from the top menu and then **Automation rule**.

   :::image type="content" source="./media/tutorial-respond-threats-playbook/add-new-rule.png" alt-text="Screenshot showing how to add a new automation rule.":::

1. The **Create new automation rule** panel opens. Enter a name for your rule.

   :::image type="content" source="./media/tutorial-respond-threats-playbook/create-automation-rule.png" alt-text="Screenshot showing the automation rule creation wizard.":::

1. **Trigger:** Select the appropriate trigger according to the circumstance for which you're creating the automation rule&mdash;**When incident is created**, **When incident is updated**, or **When alert is created**.

1. **Conditions:**
    1. Incidents can have two possible sources: they can be created inside Microsoft Sentinel, and they can also be [imported from&mdash;and synchronized with&mdash;Microsoft 365 Defender](microsoft-365-defender-sentinel-integration.md).
    
        If you selected one of the incident triggers and you want the automation rule to take effect only on incidents sourced in Microsoft Sentinel, or alternatively in Microsoft 365 Defender, specify the source in the **If Incident provider equals** condition. (This condition will be displayed only if an incident trigger is selected.)

    1. For all trigger types, if you want the automation rule to take effect only on certain analytics rules, specify which ones by modifying the **If Analytics rule name contains** condition.

    1. Add any other conditions you want to determine whether this automation rule will run. Select **+ Add** and choose [conditions or condition groups](add-advanced-conditions-to-automation-rules.md) from the drop-down list. The list of conditions is populated by alert detail and entity identifier fields.

1. **Actions:**
    1. Since you're using this automation rule to run a playbook, choose the **Run playbook** action from the drop-down list. You'll then be prompted to choose from a second drop-down list that shows the available playbooks. An automation rule can run only those playbooks that start with the same trigger (incident or alert) as the trigger defined in the rule, so only those playbooks will appear in the list.<a name="permissions-to-run-playbooks"></a>

        <a name="explicit-permissions"></a>
    
       > [!IMPORTANT]
       >
       > **Microsoft Sentinel must be granted explicit permissions in order to run playbooks**, whether manually or from automation rules. If a playbook appears "grayed out" in the drop-down list, it means Sentinel does not have permission to that playbook's resource group. Click the **Manage playbook permissions** link to assign permissions.
       >
       > In the **Manage permissions** panel that opens up, mark the check boxes of the resource groups containing the playbooks you want to run, and click **Apply**.
       >  
       > :::image type="content" source="./media/tutorial-respond-threats-playbook/manage-permissions.png" alt-text="Screenshot that shows the actions section with run playbook selected.":::
       >
       > - You yourself must have **owner** permissions on any resource group to which you want to grant Microsoft Sentinel permissions, and you must have the **Logic App Contributor** role on any resource group containing playbooks you want to run.
       >
       > - In a multi-tenant deployment, if the playbook you want to run is in a different tenant, you must grant Microsoft Sentinel permission to run the playbook in the playbook's tenant.
       >      1. From the Microsoft Sentinel navigation menu in the playbooks' tenant, select **Settings**.
       >      1. In the **Settings** blade, select the **Settings** tab, then the **Playbook permissions** expander.
       >      1. Click the **Configure permissions** button to open the **Manage permissions** panel mentioned above, and continue as described there.
       >
       > - If, in an **MSSP** scenario, you want to [run a playbook in a customer tenant](automate-incident-handling-with-automation-rules.md#permissions-in-a-multi-tenant-architecture) from an automation rule created while signed into the service provider tenant, you must grant Microsoft Sentinel permission to run the playbook in ***both tenants***. In the **customer** tenant, follow the instructions for the multi-tenant deployment in the preceding bullet point. In the **service provider** tenant, you must add the **Azure Security Insights** app in your Azure Lighthouse onboarding template:
       >    1. From the Azure Portal go to **Azure Active Directory**.
       >    1. Click on **Enterprise Applications**.
       >    1. Select **Application Type**  and filter on **Microsoft Applications**.
       >    1. In the search box type **Azure Security Insights**.
       >    1. Copy the **Object ID** field. You will need to add this additional authorization to your existing Azure Lighthouse delegation.
       >
       >    The **Microsoft Sentinel Automation Contributor** role has a fixed GUID which is `f4c81013-99ee-4d62-a7ee-b3f1f648599a`. A sample Azure Lighthouse authorization would look like this in your parameters template:
       >    
       >    ```json
       >    {
       >        "principalId": "<Enter the Azure Security Insights app Object ID>", 
       >        "roleDefinitionId": "f4c81013-99ee-4d62-a7ee-b3f1f648599a",
       >        "principalIdDisplayName": "Microsoft Sentinel Automation Contributors" 
       >    }
       >    ```

    1. Add any other actions you want for this rule. You can change the order of execution of actions by selecting the up or down arrows to the right of any action.

1. Set an expiration date for your automation rule if you want it to have one.

1. Enter a number under **Order** to determine where in the sequence of automation rules this rule will run.

1. Click **Apply**. You're done!

[Discover other ways](automate-incident-handling-with-automation-rules.md#creating-and-managing-automation-rules) to create automation rules.

### Respond to alerts&mdash;legacy method

Another way to run playbooks automatically in response to **alerts** is to call them from an **analytics rule**. When the rule generates an alert, the playbook runs.

**This method will be deprecated as of March 2026.**

Beginning **June 2023**, you can no longer add playbooks to analytics rules in this way. However, you can still see the existing playbooks called from analytics rules, and these playbooks will still run until March 2006. You are strongly encouraged to [create automation rules to call these playbooks instead](migrate-playbooks-to-automation-rules.md) before then.

## Run a playbook on demand

You can also manually run a playbook on demand, whether in response to alerts, incidents (in Preview), or entities (also in Preview). This can be useful in situations where you want more human input into and control over orchestration and response processes.

### Run a playbook manually on an alert

# [NEW Incident details page](#tab/incidents)

1. In the **Incidents** page, select an incident.

1. Select **View full details** at the bottom of the incident details pane.

1. In the incident details page, in the **Incident timeline** widget, choose the alert you want to run the playbook on. Select the three dots at the end of the alert's line and choose **Run playbook** from the pop-up menu.

    :::image type="content" source="media/investigate-incidents/remove-alert.png" alt-text="Screenshot of running a playbook on an alert on-demand.":::

1. The **Alert playbooks** pane will open. You'll see a list of all playbooks configured with the **Microsoft Sentinel Alert** Logic Apps trigger that you have access to.

1. Select **Run** on the line of a specific playbook to run it immediately.

# [Investigation graph](#tab/cases)

1. In the **Incidents** page, select an incident.

1. Select **View full details** at the bottom of the incident details pane.

1. In the incident details page, select the **Alerts** tab, choose the alert you want to run the playbook on, and select the **View playbooks** link at the end of the line of that alert.

1. The **Alert playbooks** pane will open. You'll see a list of all playbooks configured with the **Microsoft Sentinel Alert** Logic Apps trigger that you have access to.

1. Select **Run** on the line of a specific playbook to run it immediately.

---

You can see the run history for playbooks on an alert by selecting the **Runs** tab on the **Alert playbooks** pane. It might take a few seconds for any just-completed run to appear in the list. Selecting a specific run will open the full run log in Logic Apps.

### Run a playbook manually on an incident (Preview)

1. In the **Incidents** page, select an incident. 

1. From the incident details pane that appears on the right, select **Actions > Run playbook (Preview)**.  
    (Selecting the three dots at the end of the incident's line on the grid or right-clicking the incident will display the same list as the **Action** button.)

1. The **Run playbook on incident** panel opens on the right. You'll see a list of all playbooks configured with the **Microsoft Sentinel Incident** Logic Apps trigger that you have access to.

   > [!NOTE]
   > If you don't see the playbook you want to run in the list, it means Microsoft Sentinel doesn't have permissions to run playbooks in that resource group ([see the note above](#explicit-permissions)). To grant those permissions, select **Settings** from the main menu, choose the **Settings** tab, expand the **Playbook permissions** expander, and select **Configure permissions**. In the **Manage permissions** panel that opens up, mark the check boxes of the resource groups containing the playbooks you want to run, and select **Apply**.

1. Select **Run** on the line of a specific playbook to run it immediately.

You can see the run history for playbooks on an incident by selecting the **Runs** tab on the **Run playbook on incident** panel. It might take a few seconds for any just-completed run to appear in the list. Selecting a specific run will open the full run log in Logic Apps.

### Run a playbook manually on an entity (Preview)

1. Select an entity in one of the following ways, depending on your originating context:

    **If you're in an incident's details page (new version, now in Preview):**
    1. In the **Entities** widget in the **Overview** tab, find an entity from the list (don't select it).
    1. Select the three dots to the right of the entity.
    1. Select **Run playbook (Preview)** from the pop-up menu and continue with step 2 below.  
    If you selected the entity and entered the **Entities tab** of the incident details page, continue with the next line below.
    1. Find an entity from the list (don't select it).
    1. Select the three dots to the right of the entity.
    1. Select **Run playbook (Preview)** from the pop-up menu.  
    If you selected the entity and entered its entity page, select the **Run playbook (Preview)** button in the left-hand panel.

    **If you're in an incident's details page (legacy version):**
    1. Select the incident's **Entities** tab.
    1. Find an entity from the list (don't select it).
    1. Select the **Run playbook (Preview)** link at the end of its line in the list.  
    If you selected the entity and entered its entity page, select the **Run playbook (Preview)** button in the left-hand panel.

    **If you're in the Investigation graph:**
    1. Select an entity in the graph.
    1. Select the **Run playbook (Preview)** button in the entity side panel.  
    For some entity types, you may have to select the **Entity actions** button and from the resulting menu select **Run playbook (Preview)**.

    **If you're proactively hunting for threats:**
    1. From the **Entity behavior** screen, select an entity from the lists on the page, or search for and select another entity.
    1. In the [entity page](entity-pages.md), select the **Run playbook (Preview)** button in the left-hand panel.

1. Regardless of the context you came from, the instructions above will all open the **Run playbook on *\<entity type>*** panel. You'll see a list of all playbooks that you have access to that were configured with the **Microsoft Sentinel Entity** Logic Apps trigger for the selected entity type.

1. Select **Run** on the line of a specific playbook to run it immediately.

You can see the run history for playbooks on a given entity by selecting the **Runs** tab on the **Run playbook on *\<entity type>*** panel. It might take a few seconds for any just-completed run to appear in the list. Selecting a specific run will open the full run log in Logic Apps.

## Next steps

In this tutorial, you learned how to use playbooks and automation rules in Microsoft Sentinel to respond to threats. 
- Learn more about [authenticating playbooks to Microsoft Sentinel](authenticate-playbooks-to-sentinel.md)
- Learn more about [using triggers and actions in Microsoft Sentinel playbooks](playbook-triggers-actions.md)
- Learn more about 
- Learn how to [proactively hunt for threats](hunting.md) using Microsoft Sentinel.
