# YG400A-W-module-swap
How to replace the Tuya CB3S module with an ESP8266 and your own firmware

## Why
Tuya is a closed system with cloud connection to China.  
ESP8266 is (almost?) pin compatible and has open-source code initiatives that do not rely on undefined/untrusted cloud connections.

## Reverse engineering
The good news is that the Tuya site does document how to use these modules:
[LowPower SerialPortProtocol](https://developer.tuya.com/en/docs/iot/tuyacloudlowpoweruniversalserialaccessprotocol?id=K95afs9h4tjjh)  
And using an xScope-xProtoLab to analyse the 5 relevant pins, I could confirm that at least part of the behavior is described by this.
- bit2 TX
- bit3 RX
- bit4 GP9 (where GPIO15 would be on a ESP12)
- bit5 RESET
- bit6 CEN

This repo is supposed to grow over time as the reverse engineering continues and we have a full replacement strategy.

Initialy I will upload 'scope movies' and snapshots without links to them, just browse the repo

## High level observations
Because I do not want to destroy my 'evidence', I do not onboard the Tuya module yet. So, I am working with a factory clean module.  
There is a switch in the bottom plate, and this divides in two phases
- insert battery  -> this is a stage with slow signals on bit2 and bit4 and then power down on al bits after 18s.
- activate switch -> this transmits `0x55 0xaa 0x00 0x01 0x00 0x00 0x00` which [queries the product information](https://developer.tuya.com/en/docs/iot/tuyacloudlowpoweruniversalserialaccessprotocol?id=K95afs9h4tjjh#title-6-Query%20product%20information). The result is long enough to be indeed that, but decoding is not reliable yet in this stage
- after 3s -> transmit `0x55 0xaa 0x00 0x02 0x00 0x01 0x00 0x02` and get back something that does not decode well. [network status](https://developer.tuya.com/en/docs/iot/tuyacloudlowpoweruniversalserialaccessprotocol?id=K95afs9h4tjjh#title-7-Report%20the%20network%20status%20of%20the%20device) is `smartconfig configuration`
- after 2:09  -> some slow activity on bit2 and bit4 and bit3=0
- another 27s -> power down, bit4, bit6, bit5

## Challenges
### getting resolution by a fast scope and make screen recording
The nature of this debugging is that it is a long streched time scale, and yet you need resolution to e.g. detect the baudrate.
Initially I used 50ms/div which was undersampling the baud rate but managed to capture a full screen update in screen recording mode (macOS).
I then tried a faster scope but screen recording then left many gaps.
I made a second run of just the slow initial 18s which is easier to get an overview, but looses info already.
Finally I used a single trace with trigger at bit2 to capture the details of the baudrate

### Decoding 9600 baud
Somehow the xprotolab gets confused in its serial decoder mode.  
In a scope view I can confirm the header is truly `0x55 0xaa 0x00 0x01 0x00 0x00 0x00`  
BUT it gets decoded as `0x15 0x09 0x0d 0x01 0x00 0x00 0x00`  
Other decoded patterns also have this issue with the beginnings, but the ends seem to make sense according to the specs  
Using an independent serial decoder of another type might shed some light.  
AND maybe it is not too important anyhow, if we get rid of the CB3S anyway

### Other behaviour that is not serial, but still uses the CB3S TX port and the GP9 port
Much slower transitions and there could be a whole protocol there, but no idea yet. plenty of transitions to be interpreted.

## Other references
[Tasmota/Blakadder](https://templates.blakadder.com/YG400A.html)
