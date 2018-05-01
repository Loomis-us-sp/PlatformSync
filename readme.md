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
