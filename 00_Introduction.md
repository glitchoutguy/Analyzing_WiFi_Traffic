# Introduction

This SOP will cover analyzing 802.11 data with our primary tool being Wireshark. Note that this will not be an exhaustive dive into the 802.11 protocol or providing a complete understanding of wireless traffic in general. For full understanding of the specifications, parameters, values, and technologies used in 802.11, please read the IEEE whitepaper.

[IEEE 802.11be](https://ieeexplore.ieee.org/document/11090080)

At the time of this writing, the current highest standard for wireless technology is 802.11bn or "Wi-Fi 8" which introduces the concept of UHT or Ultra High Reliability which aims to improve the reliability of Wi-Fi. This iterates upon 802.11be (Wi-Fi 7) and so on.

It is important to understand these specs as they can allow you to understand what protocol is actually being used in a communication, can let you spot network interference, understand if a transmitter is transmitting to loudly, and identify exploited protocol features sch as deauthentication attacks, the KRACK WPA2 flaw, evil twins, rogue APs, and spoofing. Analysis of the management and control frames can also provide many useful details about the device conversation.


