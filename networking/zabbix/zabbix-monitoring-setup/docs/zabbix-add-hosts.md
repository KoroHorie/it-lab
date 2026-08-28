<h1>Adding a Host to Zabbix Server</h1>

<p>This section covers adding the Zabbix Agent VM to the Zabbix Server for monitoring. Zabbix provides many built-in templates that make it easy to start collecting data from hosts
without creating everything from scratch. These templates contain pre-configured items such as metrics, triggers like alert conditions, graphs and discovery rules.

For this lab, we will add the Ubuntu 24.04 Agent VM as a monitored host using the official Linux by Zabbix agent 2 template, which supports plugins and better performance.
</p>

<h3>1. Login to the Zabbix Web Interface using admin credentials</h3>

<div align="center">
  <img src="../screenshots/zabbixlogin.png"
       alt="Zabbix Login" 
       width="650">
  <p>Login the default credential of Zabbix</p>
</div>

<h3>2. Create a host on the Data Collection</h3>

<p>Navigate through Dashboard > Data collection > Hosts</p>
<p>To add a host, simply click "Create host"</p>

<div align="center">
  <img src="../screenshots/hostconfig.png"
       alt="Zabbix Login" 
       width="650">
  <p>Creation of Ubuntu Server</p>
</div>

<p>After adding the host. Zabbix will link up with the Agent VM. It takes time for it to add in its Data collection but after sometime Zabbix Server is now tracking the Agent VM</p>

<div align="center">
  <img src="../screenshots/zabbix-createdhosts.png"
       alt="Zabbix Login" 
       width="650">
  <p>Ubuntu Server Zabbix Agent successfully created</p>
</div>

<div align="center">
  <img src="../screenshots/AgentHost-SystemPerformance.png"
       alt="Zabbix Login" 
       width="650">
  <p>Ubuntu Server Dashboard Metrics</p>
</div>

<p>This now proves that the Zabbix Server can monitor the Resources and System Performance in Real-time with Ubuntu Server Zabbix Agent</p>
<p>Zabbix Servers provides visualization of the Raw data like the CPU, Memory, Storage Capacity resources by simply navigating to Monitoring > Latest data</p>

<div align="center">
  <img src="../screenshots/ubuntu-latestdata.png"
       alt="Ubuntu Zabbix Agent Resources" 
       width="650">
  <p>Filtering of Ubuntu Server</p>
</div>

<div align="center">
  <img src="../screenshots/ubuntu-resources.png"
       alt="Ubuntu Zabbix Agent Resources" 
       width="650">
  <p>Ubuntu Zabbix Agent Resources</p>
</div>
