# Loomis SafePoint PlatformSync

API (PlatformSync) documentation can be found here: https://loomis-us-sp.github.io/PlatformSync/ 

# Platform Sync Overview

This working document provides information on how Loomis PlatformSync ("API") can integrate with Customer systems.

With SafePoint by Loomis, the Smart Safe ("Safe"), integrates only to the back-end Loomis system, SafeSync. SafeSync allows Loomis to connect in a variety of ways with multiple manufacturer models and connection methods.

## Connection Methods

### Customer Internet

Uses the site’s existing internet connection allowing Safe to connect to Loomis without requiring Loomis to traverse the customer’s network

### Loomis Cellular Modem

Deployed inside the safe and offers a turn-key solution, without having to engage customer IT allowing Loomis a direct, bi-directional connection to the Safe

### VPN

Customer-to-Loomis VPN tunnel, requiring configuration, planning, and setup from Loomis and Customer network engineers in order for Loomis to traverse customer network to connect to Safe

## SafeSync

Loomis’ SafeSync platform is a multi-node highly scalable and fault tolerant platform designed to support the above connection methods while simultaneously communicating with thousands of Safes at any one time. SafeSync allows Loomis to connect to Safes once per hour (at the moment) or allows pre-configured safes to traverse Customer networks and through the internet to connect to Loomis. Data exchanged between SafeSync and the Safe is AES encrypted using a 256bit key configured on the Safe during manufacturing. 

SafeSync monitors the Safe’s sensors to ensure proper operation of all components including door sensors and validators. While the OS level functions are not exposed to any layer of the software stack, the Safe software reports any potential issues through the proprietary protocol established between Loomis and the manufacturer. This layer of abstraction allows a measure of security and protection against tampering of the lower level OS functions.

# Benefits of PlatformSync
Due to nature of connection methodology, security and integrity of the information residing in the safe, Loomis provides Customers the ability to integrate their backend systems with Loomis PlatformSync rather than individual Safes. This allows Customers to remain immune to changes to Safe software/configurations, data and transactions mutations, hardware and software failures, and take advantage of software infrastructure that Loomis has invested in since 2008 supporting thousands of customers over tens of thousands of Safes. In addition this ensures that inadvertent changes or failures on the Safe do not adversely affect Customer site POS systems.

With Loomis PlatformSync, customers can offer end points for direct integration or invoke Loomis PlatformSync APIs to gather data about their Safes. 

Below is a high level representation of how Loomis PlatformSync can integrate with Customer back-end systems.

## Serial Number
As messages are exchanged between the Loomis PlatformSync and Customer endpoints, accurately identifying individual Safes on both ends will be crucial. The most natural mechanism for this is the unique alphanumeric Serial Number assigned to the Safe by the manufacturer. The Serial Number varies in format length by manufacturer and model. 

Examples include: 

- UT01234
- CT57679
- LS2017031453

## Push API

Loomis PlatformSync can push data to customer provided end points. Customers can develop these technology agnostic end points as long as they meet the basic minimum common requirements and conform to schema for the expected set of messages. With push, Customers are responsible with ensuring the end point is up and running at all times.

- Common attributes of all customer provided end points
    - HTTPS POSTs
    - Implement agreed upon authentication and security protocols
    - XML or JSON payloads – pre configured in Loomis PlatformSync
- Event Driven or live feed
    - An Event Driven or live feed allows customers to receive data for one or more safes as the Loomis SafeSync platform collects data from the safe. For example, in the Transactions Message (detailed below) an event could be a specific type of transaction that occurs on the Safe. Customers can register the type of transaction they are interested in with Loomis PlatformSync. When any safe generates the specific transaction, Loomis PlatformSync will invoke the API and push any transactions for any safe collected since the last time the end point was invoked.
    - A Customer provided end point that supports a live feed should first and foremost account for load and scale. As the number of data points or transactions and Safes increase the higher the demand that will be placed on the Customer end point.
- Recovery, Retries and SLA
    - Loomis PlatformSync will retry any Message that returns anything other than a HTTP Status Code 200 up to three times. Any end point that fails to respond within three minutes will be considered abandoned and a retry will be attempted.
    - Loomis PlatformSync keeps track of the data that has been delivered to configured Customer end point(s) for the appropriate Message type. Depending on the configuration, Schedules or Event Driven, Loomis PlatformSync will resubmit all the data that wasn’t delivered in a prior end point invocation along with any new data that has since accumulated.
    - Loomis PlatformSync will send a notification to the Customer and Loomis Support teams for any end point that fails to accept a Message within the three retry attempts.
    - While Loomis engineers work to ensure High Availability for Loomis PlatformSync, recovery is built in to minimize down time and data integrity. Loomis PlatformSync will track data not submitted during any planned or unplanned outage and ensure delivery upon restoration of services.

## Versioning

As Loomis PlatformSync are updated with new features and fixes, changes that are not compatible with existing implementations will be deployed under a new version. Customers using an existing version will remain unaffected and can make use of updated versions when ready. Omission of the desired API version will result in the latest version of the API to be used.

If a version is deprecated or obsolete, Customers will be notified to upgrade their systems to latest versions.

## Messages
Loomis PlatformSync provide a number of messages for customers to consume that offer a variety of information. Each message type contains identifying information for a specific location and safe.

##Transactions
At the core of Loomis PlatformSync Messages, is the Transactions Message containing a list of individual transactions made by users at a specific date/time. These transactions are generated by the safe on a regular basis throughout the use of the safe. While most transaction types are generated by users, there are a few transactions that are Safe generated, but all follow the same general schema.

Below are a list of data points that are provided by Loomis PlatformSync for all transaction types:

> **THE FOLLOWING SHOULD NOT BE OUTLINED HERE, BUT IN THE ACTUAL DOCUMENTATION**

- Date/Time
    - Indicates the date and local time the transaction was performed on the safe
    - Format: ISO8601 (2017-08-01T21:20:08)
- User
    - For non-system transactions, the name of the user logged into the safe that generated the transaction (ex: cash drops)
    - The name of the user is unique across the Safe and can be created locally on the safe or through Loomis PlatformSync 
    - Format: 4-11 alphanumeric characters 
- Type
    - Unique numeric identifier to indicate the type of transaction
    - Below is a list of currently supported transaction types
        - 30 - Validated Cash Dropped into the Validator
        - 31 - Loomis Cash Courier/Pickup transaction 
        - 32 - Validated Cash Dropped into the Validator for Change Purchase 
        - 51 - Manual Drop into the Drop Vault
        - 100 - End of Day; Business Day Cutoff
    - Format: 32-bit int
- Amount
    - The dollar value of the transaction, if applicable, as some transactions do not have a value assigned. 
    - Ex: End of Day; Business Day Cutoff
    - Format: decimal
- Denominations
    - For transactions that contain amounts, a key/value pair of denominations and count of bills will be provided
    - Format: key=decimal, value=32-bit int
- Serial Number
    - An alphanumeric set of characters that uniquely identify a safe that is assigned to each safe by the manufacturer 
- Change Purchase Reference Number
    - A manual, numeric, configurable length number entered on the safe designating a certain amount of cash stored in the safe for change purchase 
    - Format: configurable up to 15 numbers
- Deposit Reference Number
    - A number entered manually into the safe each time a deposit (validated or manual) is made 
    - Format: configurable, from 4 to 15 numbers
- Register
    - A pre-determined selection made before a deposit (validated or manual) is made
    - Registers are pre-configured on the safe and only require a selection on by the user when a deposit is made
    - Registers consist of a key/value pair of a sequentially assigned number and unique name (up to 8 alphanumeric characters)
    - A safe can support up to 90 pre-configured registers
    - Format: key=sequentially assigned number, value=name
- EOD or Business Date
    - Local date/time of the business date cutoff assigned to each transaction that participated in the business day
    - Format: ISO8601 (2017-08-01T21:20:08)
- Pickup Group Id
    - A unique number assigned to a set of transactions that occurred within the safe Loomis Courier Pickup period
    - All transactions that occur on the safe after Loomis Courier Pickup up to and including the next Loomis Courier Pickup will have the same unique number
    - Format: 64-bit integer

## Push Endpoint Requirements
When utilizing the Push mechanism for the Transactions Message, Customers will be responsible for implementing an end point that implements a specific schema.
Customer provided end point(s) must
- Accept HTTPS POST only
- Accessible over Internet
- XML or JSON – pre-determined and stored in Loomis PlatformSync
- Implement entire agreed upon security protocol 
- Accept a list of transactions with data points noted above
    - List may contain more than one Safe (serial number)

Loomis PlatformSync will invoke a set of pre-configured end points. A payload posted to the end point(s) can contain one or more safes. Depending on the Customer configuration, the end point(s) will either be invoked on schedule(s) or event driven (one or more transactions are retrieved). 

Any set of transactions for any Safe that have not been sent to the Customer end point(s) will be delivered upon the next event or schedule. Loomis PlatformSync will internally track which transactions for which Safe have been posted to the Customer end point(s). In the event of an outage, either minor or extended, when Loomis PlatformSync comes back on line, any pending transactions that have yet to be posted will be sent to the Customer end point(s).

Loomis PlatformSync will ensure that duplicate transactions for any safe are not posted.

A “starting date” will be configured and stored in Loomis PlatformSync that will be used as the initial reference point from which all transactions from all Safes will be posted to Customer end point(s). At any time a customer may choose to “reset” the “starting date”, however Loomis PlatformSync will not be responsible for any duplicate transactions that may result as part of the “reset”.

## Security

TBD – Secured with SSL Certificates and API keys for both Push and Pull models

## Environment and SLAs

Loomis PlatformSync are hosted and operated by Loomis within a Loomis managed data center in Houston, TX. The entirety of the infrastructure required to operate Loomis PlatformSync and all backend systems is provisioned and maintained by Loomis system, network and software engineers. 

While Loomis engineers have built redundancy and fault tolerance at all levels from servers to network equipment and the software systems, unforeseen events may occur that result in a loss of uptime as it relates to Loomis PlatformSync. 

Loomis Support personnel will work to ensure Customers are notified of any downtime that may occur as a result of planned or emergency outages. In addition Loomis PlatformSync provide recovery capabilities to ensure loss of data does not occur and the process resumes as normal.

## Changes and Updates

Loomis reserves the right to make modifications and enhancements to the Loomis PlatformSync, to support customer requests, fix issues and offer new features.

These enhancements will be available transparently to customers on compatible version of Loomis PlatformSync. If a set of changes are not compatible with the previously available version, Loomis will provide a new version that will be available to customers as needed.

Changes to Loomis PlatformSync will be communicated with customers prior to release to ensure awareness and compatibility. 
