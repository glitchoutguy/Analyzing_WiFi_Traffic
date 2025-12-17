# Understanding Wireless Conversations

A wireless conversation is made up of several components and frame exchanges that negotiate how a wireless client connects to a wireless access point (WAP). These steps can be defined under the following 7 categories:

1. Discovery
2. Authentication
3. Association
4. Security & Key exchanges
5. Data Transfer
6. Roaming / Reassociation
7. Disassociation / Deauthentication

## Discovery

The discovery phase of the wireless conversation involves 3 major IEEE 802.11 frame types:
Beacons (from WAPs)
Probe Requests (from Clients)
Probe Responses (from WAPs)

Within Wiresahrk, this information can be found under the IEEE 802.11 <type> frame and Wireless Management header.

## Beacons

`wlan.fc.type_subtype == 0x0008`

The purpose of a beacon frame is to advertise a WAPs service set identifier (SSID) to wireless clients. A beacon frame contains various metadata and parameters, which are found under IEEE 802.11 Wireless Management. Important information can be found in the order of:

### Timestamp

### Capabilities Information 

Note that according to the 802.11 spec, in order for a WAP to advertise an SSID, the ESS Capabilities bit has to be set to 1. You can view this with the BPF `wlan.fixed.capabilities.ess == True` With this bit set to 1 the device is is infrastructure mode and the SSID belongs to an infrastructure network. If the ESS capabilities bit is 0, but the IBSS bit is 1, it means that the device is part of an ad-hoc (peer-to-peer) network. If both are 0, then the advertisement is unusual and could indicate a misconfigured AP, nonstandard device, or special meaning frame.

### SSID parameter set

Defines the SSID name string and length. Note that if the length is set to 0, the SSID is hidden.

### DS parameter set

Denotes the channel the beacon frame was sent from.

### Country Information 

Note that this will include the number of channels and maximum transmit power level of the device. The number of channels is important as different countries utilize different numbers of channels. In the United States, we use channels 1-11 for 2.4HGz networks, in the EU they use channels 1-13 and in Japan 1-14. If a device within the US is operating on a non-standard channel, it is violating FCC regulation and needs to be investigated and remediated. It is worth noting that in the US, there are 3 non-overlapping channels: 1,6 11. These channels are used for data transfer, but in the EU and Japan, there are actually 4 possible channels: 1, 6, 11, 13. You can see why this is important as there is an entire extra reliable data channel that exists outside of US specifications. If an adversary were to utilize configurations such as this, they could potentially open hidden channels inside of a network where only channels 1-11 would be monitored. Additionally, extra wireless networks could be configured to operate only on those extra channels, providing hidden backdoors into a network.

| Channel # | Center Frequency (MHz) | US (FCC) | Europe (ETSI) | Japan (MIC)      |
| --------- | ---------------------- | -------- | ------------- | ---------------- |
| 1         | 2412                   | ✅        | ✅             | ✅                |
| 2–11      | 2417–2462              | ✅        | ✅             | ✅                |
| 12        | 2467                   | ❌        | ✅             | ✅                |
| 13        | 2472                   | ❌        | ✅             | ✅                |
| 14        | 2484                   | ❌        | ❌             | ✅ (802.11b only) |


Within Wireshark, this is what a Beacon frame from a rogue or out of spec WAP may look like:
```
Frame 45: 162 bytes on wire (1296 bits)
IEEE 802.11 Beacon frame, SN=115, FN=0, Flags=...
    Type/Subtype: Beacon (0x08)
    Destination: Broadcast (ff:ff:ff:ff:ff:ff)
    Source: 00:11:22:33:44:55 (RogueAP)
    BSS Id: 00:11:22:33:44:55 (RogueAP)
    Capability Information: ESS+Privacy
    Tagged parameters:
        SSID parameter set: RogueNetwork
        Supported rates: 1(B), 2(B), 5.5(B), 11(B)
        DS Parameter set: Current Channel: 13
```

Here is what a Probe request from a US Client would look like:
```
Probe Request, SSID: Broadcast
    Supported rates: 1(B), 2(B), 5.5(B), 11(B), 6, 9, 12, 18, 24, 36, 48, 54
    Extended Supported Rates: 6, 9, 12, 18, 24, 36, 48, 54
```

Note that there is no channel information. The client will only probe channels 1-11, it never transmits on channels 12-13 so it does not discover the AP. If a rogue client were present, it would be able to send a probe request for a device in channel 13, which means that it would receive a probe response from the AP.

### TPC Report Transmit power

Specifies the transmit power of the AP radio. Like channels, the maximum that this may be is determined by country. This information can be found in the table below:

| Region / Regulator                     | Max Tx Power (EIRP) | Notes                                                                                            |
| -------------------------------------- | ------------------- | ------------------------------------------------------------------------------------------------ |
| **United States (FCC)**                | **30 dBm (1 W)**    | Up to 4 W EIRP with directional antennas (point-to-point links). Common Wi-Fi APs run far lower. |
| **European Union (ETSI)**              | **20 dBm (100 mW)** | Strict limit across 2.4 GHz; outdoor use sometimes further restricted.                           |
| **Canada (ISED)**                      | **30 dBm (1 W)**    | Similar to FCC; must follow RSS-247.                                                             |
| **Australia / New Zealand (ACMA/RSM)** | **36 dBm (4 W)**    | Higher outdoor power permitted in 2.4 GHz ISM band.                                              |
| **Japan (MIC)**                        | **20 dBm (100 mW)** | Channel 14 allowed but only 802.11b DSSS at 11 Mbps.                                             |
| **China (MIIT)**                       | **20 dBm (100 mW)** | Aligned with ETSI-like rules.                                                                    |
| **India (WPC)**                        | **30 dBm (1 W)**    | License-exempt for 2.4 GHz ISM use.                                                              |
| **Brazil (ANATEL)**                    | **30 dBm (1 W)**    | Similar to FCC/Canada.                                                                           |


### RSN Information

Defines the cipher suites that the client and AP will use in various situations. Secure implementations will utilize AES (CCMP) and group management cipher suites (WPA3/802.11w) that protects against broadcast management frames like beacons and deauths.

### AP Channel Report

Defines the device operating class and the channels the device will use.

### HT Capabilities

High-Throughput introduced in 802.11n that defines many physical wavelength settings, many of which determine real-world performance between an AP and clients.

### Vendor Specific 

Vendors often want to add their own custom functionalities. Every vendor specific information Element (IE) follows the same format.

| Field                                        | Length   | Description                                                                   |
| -------------------------------------------- | -------- | ----------------------------------------------------------------------------- |
| **Element ID**                               | 1 byte   | Always `221` (0xDD) for Vendor Specific                                       |
| **Length**                                   | 1 byte   | Length of the rest of the IE                                                  |
| **OUI (Organizationally Unique Identifier)** | 3 bytes  | Identifies the vendor (from IEEE’s registry). Example: `00:50:F2` = Microsoft |
| **OUI Type**                                 | 1 byte   | Vendor-defined type field                                                     |
| **OUI Subtype** (optional)                   | 1 byte   | Further breakdown                                                             |
| **Vendor Data**                              | variable | Proprietary data or extensions                                                |


## Probe Requests

`wlan.fc.type_subtype == 0x0004`

Probe requests are 802.11 management frames sent by a client to discover wifi networks. They function much like Beacons from WAPs but reversed in the sense that the client is broadcasting instead. Nearly all of the above information can be applied to the dissection of wireless client probe requests, minus the information specific to APs. Just know that wireless clients and APs are constantly broadcasting their availability and capabilities.

Generally probe requests will be sent out by clients with a wildcard SSID, however many wireless devices will also attempt to find networks they have been connected to in the past. For instance a smart phone that had been connected to a home network may also attempt to discover new networks. This means the phone will be sending out both types of probe requests and generating traffic that may indicate certain SSIDs exist in an environment that in reality are nowhere nearby. So how can we differentiate which APs are actually nearby and which are far away? The answer is Probe Responses.

## Probe Responses

`wlan.fc.type_subtype == 0x0005`

A probe response is a 802.11 management frame sent in response to a wireless clients probe request. The purpose of this frame is to let the client know that an AP is available and to provide its network capabilities. Probe requests are extremely similar in structure and information, and they do in fact contain much of the same data, but there are a few key differences:

| Feature       | Beacon                   | Probe Response                                                              |
| ------------- | ------------------------ | --------------------------------------------------------------------------- |
| Initiator     | AP                       | AP (in response to client)                                                  |
| Transmission  | Periodic (every \~100ms) | Only after probe request                                                    |
| Destination   | Broadcast                | Unicast (client) or broadcast                                               |
| Same content? | Mostly                   | Very similar to beacon, but can include extra IEs for the requesting client |


This is arguably the most important type of discovery frame in a scenario where you must discover or hunt for rogue / undocumented transmitters. Probe requests are only sent in response to probe requests, thus letting you know that the access point exists and is actively responding to requests.


## Authentication

`wlan.fc.type_subtype == 0x0b`

An Authentication Request frames purpose is to start the authentication process between a client and an AP. There are 2 major ways that a device will authenticate to an AP. One is in an open system (no password) and the other is with a shared key. Almost all modern networks use the open system and perform encryption at a later stage by exchanging WPA2 / WPA3 keys which ultimately determine if a client can begin data transfer or not. Legacy systems like WEP use a shared key in a system that is vulnerable and exploitable.

### Open system

In an open system, the client will send an authentication request to the AP that looks something like this:

```
Frame 20: Authentication, Open System
    Source: 00:11:22:33:44:55
    Destination: 00:aa:bb:cc:dd:ee
    Authentication Algorithm: Open System (0)
    Sequence Number: 1
    Status Code: 0
```

The AP will send an authentication response with code 0 as long as the client is allowed to connect or a non-zero number if access is denied.

```
Frame 21: Authentication, Open System
    Source: 00:aa:bb:cc:dd:ee
    Destination: 00:11:22:33:44:55
    Authentication Algorithm: Open System (0)
    Sequence Number: 2
    Status Code: 0 (successful)
```

## Association

`wlan.fc.type_subtype == 0x00 || wlan.fc.type_subtype == 0x01`

Association is the step where a wireless client officially joins a Wi-Fi network. After authentication succeeds, the client and AP agree RSN / encryption settings, association ID (unique identifier for the client on the AP), and other session parameters.

The association process is completed over 2 frames:

1. Association Request (0x00)
2. Association Response (0x01)

The client first sends an association request to the AP, then the AP will send a response to the request, finalizing the connection.

An example Association Requests looks like this:
```
Frame 30: Association Request
    Destination: 00:aa:bb:cc:dd:ee (AP)
    Source: 00:11:22:33:44:55 (Client)
    SSID: MyWiFi
    Supported Rates: 1,2,5.5,11,...54 Mbps
    HT Capabilities: 20 MHz, Short GI
    RSN Information: WPA2-PSK, CCMP
```

An example Association Response looks like this:
```
Frame 31: Association Response
    Destination: 00:11:22:33:44:55
    Source: 00:aa:bb:cc:dd:ee (AP)
    Status Code: Successful (0)
    AID: 1
    RSN Information: WPA2-PSK, CCMP
```

## Security and Key exchanges

`eapol` or `wlan.fc.type == 2 && eapol` to specifically show WPA key messages.

Once a client is associated with a WAP, the client and AP can begin the process of a 4-way handshake. The handshake is where the devices will generate and install encryption keys necessary for encrypting data transfer in the next step. There are two types of keys that are generated during this step. A Pairwise Transient Key (Unicast) and a Group Temporal Key (Broadcast / Multicast).

Note that during the handshake **Passwords / Passkeys are not sent over the air**. Encryption keys are derived from data known to both the AP and client and calculated to ensure they match. The "Wi-Fi password" is never sent to or from the client or AP, it is the pre-shared key (PSK) upon which both calculate their encryption key. If this does not match, encrypted communications and data transfer are not possible between the client and AP. In enterprise environments, it is more likely that the network will use RADIUS/802.1X authentication mechanisms. The mechanism by which the client authenticates during 802.1X is more complex and occurs before the 4-way handshake, but will not be covered as this conversation does not occur fully over the air.

### Diagram of Flow

```
AP                                   Client
 |---------(1) ANonce ---------------->|
 |<--------(2) SNonce + MIC -----------|
 |---------(3) GTK + MIC -------------->|
 |<--------(4) MIC --------------------|

```

### Handshake Frame 1

During message 1, the AP will send a value called the ANonce (Authenticator Nonce) and a replay counter value. A nonce is a randomly generated value that is used only once in cryptography. They are used to prevent replay attacks where an attacker intercepts communications and retransmits it to gain unauthorized access. The replay counter is another mechanism meant to deter retransmitting messages. The replay counter increases by one for each communication and will discard any messages that come at unexpected times such as number 1 when message 2 has already been processed and it is now expecting replay value 2.

```
802.1X Authentication, Key (Message 1 of 4)
    Key Descriptor Type: IEEE 802.11
    Key Information: Key MIC=0, Secure=0, Message 1/4
    Key Nonce (ANonce): 9f:1a:cd:5e:44:9b:12:...
    Replay Counter: 1
```

### Handshake Frame 2

During message 2, the client sends back an SNonce value, a Key MIC, and the replay counter. Like the ANonce, the Supplicant Nonce is a randomly generated value which will eventually be used to calculate the PTK used by both the client and AP. The Key Message Integrity Code is a cryptographic MIC that proves the client knows the PMK (a value generated during 802.1X authentication which was no covered in this SOP). If the MIC fails verification, the handshake is aborted. The replay counter that the client sends must match the one expected by the AP. Message 1 sets the replay counter to 1, so the client must send back a message that also has a replay value of 1.

```
802.1X Authentication, Key (Message 2 of 4)
    Key Descriptor Type: IEEE 802.11
    Key Information: Key MIC=1, Secure=0, Message 2/4
    Key Nonce (SNonce): 34:af:77:bb:c3:8d:...
    Key MIC: 23:89:bc:17:d1:...
    Replay Counter: 1
```

### Handshake Frame 3

During Message 3, the AP will send the GTK for install and the MIC with the purpose of confirming the client has generated and installed the PTK.

```
802.1X Authentication, Key (Message 3 of 4)
    Key Descriptor Type: IEEE 802.11
    Key Information: Key MIC=1, Secure=1, Install=1, Message 3/4
    Key Nonce (ANonce): 9f:1a:cd:5e:44:9b:12:...
    Key MIC: 67:ac:1e:94:cd:...
    GTK KDE: Group Temporal Key (encrypted)
    Replay Counter: 2
```

### Handshake Frame 4

During message 4, the client sends the MIC back to the AP and confirms that it has installed the keys. After this frame is sent, a secure session starts and the devices can begin transmitting encrypted data.

```
802.1X Authentication, Key (Message 4 of 4)
    Key Descriptor Type: IEEE 802.11
    Key Information: Key MIC=1, Secure=1, Message 4/4
    Key MIC: 92:1c:4e:7f:bc:...
    Replay Counter: 2
```

## Deauthentication

`wlan.fc.type_subtype == 0x0c`

Deauthentication will be the last section of the wireless conversation that is covered as it is one of the most abused management frames in IEEE 802.11 and is often used by attackers to gain access to a wireless network. 

A deauthentication frame is used when a client or AP leaves a network and wants to disconnect gracefully. Additionally, a device may send a deauth if it is shutting down or a client is being removed. In an authentication context, if a client fails security checks like having a bad MIC, invalid handshake, or wrong PMK, the AP may deauthenticate it. A final case that it may be used is when a client is roaming and must move from one AP to another and must first deauth the old AP before associating with the new one.

Generally a deauthentication frame will contain client information (which client is being deauthed) and the reason code explaining why.

```
Frame 235: 26 bytes on wire (208 bits)
IEEE 802.11 Deauthentication
    Type/Subtype: Deauthentication (0x0c)
    Destination: 88:32:9b:7c:21:aa
    Source: 34:12:98:dd:ef:33
    BSS Id: 34:12:98:dd:ef:33
    Reason Code: 3 (Station is leaving (or has left) IBSS or ESS)
```

The reason why deauthentication frames are abused by attackers so often is the fact that in WPA2, deauth frames are not protected by default. They are neither encrypted nor integrity-checked meaning that anyone can spoof them. In a deauth attack, an attacker spams fake deauth frames to disconnect users so that they can use tools like airmon-ng to capture parts of the handshake so they can offline brute force the PSK. If an attacker is conducting an evil-twin attack, the deauths may force clients to connect to a fake AP instead. Deauth attacks can also be used in a more traditional DoS attack, preventing users from using the network. This attack is less prevalent in WPA3 which utilizes 802.11w (Protected management frames, PMF) to encrypt and sign deauth frames.


