<h1>Setting up Slack Notification in Zabbix</h1>

<p>This section covers integrating Zabbix with Slack so that alerts are sent directly to a dedicated Slack channel.

The integration uses Zabbix's Webhook media type, which sends formatted messages to Slack whenever a problem is detected
or resolved

Prerequisites
- A Slack workspace
- A dedicated channel in Slack workspace where Zabbix can send a message in it.
- Zabbix Server with internet access
</p>

<h1>Create a Slack App</h1>

<p>The goal of creating a Slack App is to allow Zabbix to send a real-time notifications directly into a specific Slack channel. This channel should
exist inside your Slack Workspace.
</p>

<div align="center">
  <img src="../screenshots/slackapp.png"
       alt="Slack App" 
       width="650">
  <p>Created Bot for Slack that can send messages on the channel using its API</p>
</div>

<p>When Zabbix sends a request to Slack, it uses its API to validate the bot token and it checks permissions, ensuring that it has chat:write permission and posts the message into the specified channel as the Zabbix Alerts bot.</p>

<div align="center">
  <img src="../screenshots/Slack-writeperm.png"
       alt="Slack App" 
       width="650">
  <p>Slack chat write permission for it allow post a message to the channel</p>
</div>

<div align="center">
  <img src="../screenshots/Slack-bot.png"
       alt="Slack App" 
       width="650">
  <p>Slack Bot OAuth Token which will be used by Zabbix to authenticate to Slack and get authorized to send the message to the Slack Workspace channel</p>
</div>

<h1>Configure Media Type on Zabbix</h1>

<p>Zabbix can be configured to send an alerts if there are issues that has been detected. It can use different Media Type
which includes Slack.

Configuring Slack on Zabbix can be found and navigated on Alerts > MediaType
</p>

<div align="center">
  <img src="../screenshots/ZabbixMediaType.png"
       alt="Zabbix MediaType" 
       width="650">
  <p>Here we will see several media types that we can use. We will have to enable Slack and configure it</p>
</div>

<div align="center">
  <img src="../screenshots/SlackConfig.png"
       alt="Zabbix Slack config" 
       width="650">
  <p>Slack Configurations</p>
</div>

<h1>Create an Alert Trigger Action and Operation using User-based</h1>

<p>After configuring Slack on the Media Type. We will create a Trigger Action. The Trigger Action tells Zabbix what to do when a trigger fires.
In Alerts > Trigger Actions, here we can create an Action and add a condition using Trigger Severity. This checks the severity level of the trigger
</p>

<div align="center">
  <img src="../screenshots/ZabbixTriggerActions.png"
       alt="Zabbix Trigger Action" 
       width="650">
  <img src="../screenshots/ZabbixTriggerOperations.png"
       alt="Zabbix Trigger Operations" 
       width="650">
  <p>Action uses Trigger Severity and the condition is Greater than or Equal to Warning. This shows that once Zabbix detects a Warning level. It will
  Trigger this action and for the Operations, we need to specify which user will be the one to send a message to the Slack Workspace.</p>
  <p>For this Lab, we can use the Zabbix Administrator but for security practice we can create a least privilege user</p>
</div>

<h1>Result</h1>

<div align="center">
  <img src="../screenshots/Slack Messages.png"
       alt="Zabbix Alerts posts a Notification" 
       width="650">
  <p>Zabbix Alerts posts a Notification</p>
</div>
