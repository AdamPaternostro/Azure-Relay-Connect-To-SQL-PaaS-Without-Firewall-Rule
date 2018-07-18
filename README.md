# Azure-Relay-Connect-To-SQL-PaaS-Without-Firewall-Rule

## Overview
Using Azure Relay for allowing access to SQL PaaS (and CosmosDB, and anything you want) without whitelisting your IP address in SQL Firewall.

Customers have asked for a way where they can connect to SQL PaaS, but do not want to whitelist all their developers machines IP addresses.  
This is important since developers work from homes, airplanes, hotspots, etc.
The goal is to allow the developer to securly connect to SQL PaaS without whitelisting their IP.

I saw a feature about Hybrid Connections that I through would be great, but it is very much tied to Azure App Services.
See:  https://azure.microsoft.com/en-us/resources/videos/azure-app-service-with-hybrid-connections-to-on-premises-resources/

I talked with Clemans, who was working on a more generic solution and he has implemented the below.
https://github.com/clemensv/azure-relay-bridge

## Goal
<<IMAGE>>

## Configuration
1. Azure Relay
   - In Azure create a new Azure Relay
   - Name: sqldatabaserelay (you have to select a unique name for this)
   - Region: Same region as your SQL / CosmosDB / etc.
   - <<sqldatabaserelay>>

2. In the newly created relay click on "Hybrid Connections"
   - Click "+ Hybrid Connection" and name it: sqlhybridconnection
   - <<createhybridconnection>>

3. Click on the Hybrid Connection.
   - Click on Shared access policy
   - Create a policy and name it: sqlhybridpolicy
   - Select Listen and Send.
   - Press Create.
   - <<sqlhybridpolicy>>

4. Click on the policy and copy the Primary Connection String
   - e.g. Endpoint=sb://sqldatabaserelay.servicebus.windows.net/;SharedAccessKeyName=sqlhybridpolicy;SharedAccessKey=<<REMOVED>>;EntityPath=sqlhybridconnection

5. Create an Azure VM (aka "Reciever")
   - I created a Windows Server 2016 VM
   - The name is not important.
   - The region should be in the same region as your database.
   - The machine must be on the VNET you want to trust to your SQL Database.  My VNET is named: aRelayDest-VNET
   - Login into the VM (turn off IE enhanced security).
   - Download the MSI from here: https://github.com/clemensv/azure-relay-bridge/releases
   - Install the software.

6. Now create a SQL Database in Azure
   - You can use an existing one or create a new one.
   - If you using an existing one the import step here is to set the firewall rules.
   - My database server is: hdihiveserver.database.windows.net
   - Click on the Firewall rules and set the following
   - Allow Access to Azure Services: You can choose On or Off (I did Off since I just want my VNET to access)
   - Client IP Addresses: I left blank since I am trusting my VNET.
   - Click "Add existing virtual network".  Select the Virtual Network your Receiver VM is on
   - <<sqlserverfirewall>>

 7. Client machine
   - You now need to configure your machine (or any other client)
   - Download the MSI from here: https://github.com/clemensv/azure-relay-bridge/releases  
   - Install the software.

## Running the software
Client
```
cd "C:\Program Files\Azure Relay Bridge"
azbridge -L 127.0.5.1:1433:sqlhybridconnection -x Endpoint=sb://sqldatabaserelay.servicebus.windows.net/;SharedAccessKeyName=sqlhybridpolicy;SharedAccessKey=<<REMOVED>>;EntityPath=sqlhybridconnection
```

Receiver
```
cd "C:\Program Files\Azure Relay Bridge"
azbridge -R sqlhybridconnection:hdihiveserver.database.windows.net:1433 -x Endpoint=sb://sqldatabaserelay.servicebus.windows.net/;SharedAccessKeyName=sqlhybridpolicy;SharedAccessKey=<<REMOVED>>;EntityPath=sqlhybridconnection
```


