# Getting Started Guide

HoustonMesh Getting Started Guide

Adapted to Markdown from the [original Google Doc](https://docs.google.com/document/d/17etc1riws-fjZ4gc8lAcYkFH1q6GPE21ZRCTNr-wBJo/mobilebasic)

***THIS GUIDE IS A WORK IN PROGRESS! ALL STEPS BELOW MAY NOT BE TESTED AND VERIFIED YET!***

This guide assumes you have a freshly wiped and flashed node. [a] [b] 

## Device Setup

**Set Name**

Set your long name to whatever you like, but your short name can only be 4 characters.

- **Android**: Radio configuration > User > Long name / Short name
- **iOS**: Settings > User > Long Name / Short Name
- **Python**: meshtastic --set-owner “My Long Name” --set-owner-short Name

**Set Region (US)**

- **Android**: Radio configuration > LoRa > Region = United States
- **iOS**: Settings > LoRa > Region = United States
- **Python**: `meshtastic --set lora.region US`

**Set Frequency Slot (20)**

For US LongFast, the default channel slot is 20. By default your node may already show 20 (or 0 on some clients), but if you do not manually set this to 20 and save, this value can automatically get changed if you change the primary channel (channel 0) name and you do not want this to happen or you will lose connection to others on the mesh.

- **Android**: Radio configuration > LoRa > Frequency Slot = 20
- **iOS**: Settings> LoRa > Frequency Slot = 20
- **Python**: `meshtastic --set lora.channel_num 20`

**Increase Hop Limit (optional)**

While not usually recommended on very dense Meshtastic networks, you may wish to increase your hop limit to 4 or 5.

Unlike the very dense meshes in the UK and US bay area, Houston’s physical mesh can be fairly sparse depending on your part of town and how many neighbors your node can reach. Also our channel utilization is usually relatively low, so increasing hop limit may improve how many nodes you can see.

When using MQTT each MQTT gateway is a hop; this means two RF meshes each using a MQTT gateway 2 of the 3 default hops would be used by crossing over MQTT from one mesh to the other.

- **Android**: Radio configuration > LoRa > Hop limit = 4
- **iOS**: Settings > LoRa > Number of hops = 4
- **Python**: `meshtastic --set lora.hop_limit 4`

**Enable OK to MQTT (optional)**

Since v2.5+ your node’s messages will not traverse MQTT by default. While you may choose to leave this disabled, understand that you may receive messages from a user or in a channel who will not be able to receive the messages you send back. If you prefer an RF only approach then you probably want to also enable Ignore MQTT (`lora.ignore_mqtt`) in addition to leaving OK to MQTT disabled.

- **Android**: Radio configuration > LoRa > OK to MQTT = On
- **iOS**: Settings > LoRa > Ok to MQTT = On
- **Python**: meshtastic --set lora.config_ok_to_mqtt true

**Set Nodeinfo Broadcast Interval (optional)**

Default is 10800 secs (3 hours), you can optionally lower this to 3600 (1 hour)

- **Android**: Radio configuration > Device > Nodeinfo broadcast interval = 3600
- **iOS**: Settings > Device > Node Info Broadcast Interval = One Hour
- **Python**: meshtastic --set device.node_info_broadcast_secs 3600

**Set Timezone (Central)**

- **Android**: Radio configuration > Device > POSIX Timezone = CST6CDT,M3.2.0/2:00:00,M11.1.0/2:00:00
- **iOS**: Device > Settings > Device > Time Zone = CST6CDT,M3.2.0/2:00:00,M11.1.0/2:00:00
- Python: meshtastic --set device.tzdef CST6CDT,M3.2.0/2:00:00,M11.1.0/2:00:00 

**Leave Role CLIENT**

<u>**You should not run repeater/router modes**</u> unless you have a dedicated node placed very high (400+ feet) with reliable power. Introducing poorly placed repeater/routers can cause more harm than good to the overall mesh.

Nodes running client mode will still repeat messages from other nodes, however if you have several nodes and a dedicated outdoor node (solar/roof/attic) then you may want to consider running client mute mode on indoor stationary nodes that do not need to rebroadcast messages that generate additional mesh traffic with little to no benefit.



## Channel Setup

It is up to you which channels to add to your node and in which order. Included below are some common channels used in the region as well as some important notes to be aware of regarding channels.

| **Channel Name** | **PSK**                    | **Key Length** | **Description**                                  |
| ---------------- | -------------------------- | -------------- | ------------------------------------------------ |
| Houston          | `DgntMessWithTexas0000w==` | 128-bit        | Houston region (may/can be MQTT connected)       |
| Texas            | `DgntMessWithTexas0000w==` | 128-bit        | Greater Texas region (may/can be MQTT connected) |
| LongFast         | `AQ==`                     | 8-bit          | RF-only default channel (see note below*)[c]     |



**Picking your Channel 0 (aka primary channel or default channel)**

This channel slot is unique in that it is the only channel that a node will send periodic/interval based location or optional telemetry (battery, temperature, other sensors) updates to automatically. While location can be enabled on secondary channels (channels 1-6), your position will only be sent to those channels if requested by another node on that same channel; it will not be sent automatically based on timer or smart location.

Some people prefer to set Houston as their primary, some prefer to keep LongFast as their primary, some prefer to use their own custom channel name and PSK for their primary channel that is private to them, and then placing other channels as secondary.

***Important note**: When changing channel 0 from the default LongFast channel name, your node may also automatically change the LoRA Frequency Slot (lora.channel_num) to something other than the default of 20. The exact number it changes to is derived from an algorithm and the channel name, but what is important is that if it is no longer 20 you will no longer be on the same frequency as the rest of us and are most likely on a quiet and lonely mesh of 1. Set this back to 20 (see Device Setup section) to restore connectivity to the larger mesh.*

**Add a Channel**

- **Android**: Radio configuration > Channels > *tap plus (+) button*
- **iOS**:
- **Python**: `meshtastic --ch-add 1 --ch-set name Houston --ch-set psk base64:DgntMessWithTexas0000w==`

**Edit a Channel**

- **Android**: Radio configuration > Channels > *tap on a channel to edit*
- **iOS**:
- **Python**: `meshtastic --ch-index 0 --ch-set name Houston --ch-set psk base64:DgntMessWithTexas0000w==`

1. Enter Channel Name, PSK, and (if needed for iOS) Key Length
2. Leave uplink enabled and downlink enabled off (unless this node will be a MQTT gateway for this channel)
3. Optionally, to share your location with the mesh turn Position enabled on and set precision
4. You can obfuscate the location your node reports to the channel using precision. This way if desired you can allow others to see your approximate location and not publish your exact location to public or semi-public channels for privacy reasons.
5. Tap Save
6. Once finished adding/editing/reordering channels, tap Send to save the new channel list to the node



## **MQTT Setup**

**Set MQTT root topic to `msh/US/TX`**

- **Android**: Module configuration > MQTT > Root topic = `msh/US/TX`
- **iOS**: Settings > MQTT > Root Topic = `msh/US/TX`
- **Python**: `meshtastic --set mqtt.root msh/US/TX`

**Enable MQTT**

- **Android**: Module configuration > MQTT > MQTT enabled = On
- **iOS**: Settings > MQTT > Enabled = On
- **Python**: `meshtastic --set mqtt.enabled true`

**Enable client proxy (optional, intended for Bluetooth nodes)**

If trying to use MQTT directly on a mobile node that is not connected to Wi-Fi, you can enable this to allow the node to use your phone’s internet via Bluetooth to connect to MQTT. For home use, a Wi-Fi connection or a dedicated MQTT gateway node is usually the ideal use case.

- **Android**: Module configuration > MQTT > Proxy to client = On
- **iOS**: Settings > MQTT > MQTT Client Proxy On
- **Python**: `meshtastic --set mqtt.root proxy_to_client_enabled true`

**Enable uplink/downlink per channel (optional)**

You can optionally allow an MQTT connected node to uplink/downlink messages from select channels between RF and MQTT, effectively sharing the MQTT connection with other nodes without Wi-Fi or Bluetooth. A dedicated node for this is called a MQTT Gateway or a MQTT Relay.

*You should <u>**never**</u> downlink default channels such as LongFast.*

You should never enable JSON for MQTT unless it’s to a private MQTT server you control. Encrypted Meshtastic messages are still sent in the clear over MQTT when using JSON.

As of late summer 2024 the public Meshtastic MQTT server will rewrite all messages sent to/over a channel using a default PSK of AQ==. This can make MQTT not very useful unless running directly on each node since no messages are retransmitted past an MQTT node.

*Uplinking = when a node retransmits a message received from RF to MQTT
Downlinking =  when a node retransmits a message received from MQTT to/over RF*

- **Android**: Radio configuration > Channels > *select a channel* > Uplink/Downlink enabled = On
- **iOS**: Settings > Channels > *select a channel* > Uplink/Downlink Enabled = On
- **Python**: meshtastic --ch-index 1 --ch-set uplink_enabled true --ch-set downlink_enabled true

**Leave default Address `mqtt.meshtastic.org`**

**Leave default Username `meshdev`**

**Leave default Password `large4cats`**





*[a] Verified tested and working*

*[b] iOS + firmware 2.5.5*

*[c] Add note about MQTT and LongFast on same node?*
