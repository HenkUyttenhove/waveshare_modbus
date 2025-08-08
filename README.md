# Home Assistent with WaveShare 16 channel relay with Modbus TCP to RS485 RTU

### Objective of this project
Domotica is here to stay and [Home Assistant] (https://www.home-assistant.io/) has further lowered the need for technical specialists.
There are different vendors on the IoT market delivering Wifi/Zigbee based solutions and I also have already deployed a Sonoff network.
However, the need for wireless signals and the dependency on the Sonoff online gateway to support integration with platforms as Google Home and Home Assistent is a limitation.

While I was surfing on the Internet, I noticed the very cheap Waveshare relay systems.  These however are running on modbus TCP or modbus RS485.  
The modbus TCP may sound as the easiest platform but because I'm also looking at integration of solar panels and heat pump, I went for the RS485 to learn more about this.
AS RS485 is a special 2-wire protocol (A+B and ground), you need an additional gateway.  For this gateway, you can take the older Waveshare RS485 LAN gateway, but I took the newer Waveshare RS485 Wifi-LAN device.

![alt text](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/overview.jpg)


### Choice of devices
[The 16-channnel relay] (https://www.waveshare.com/modbus-rtu-relay-16ch.htm) is able to support 16 channels of 10 Amp/240V was is very impressive.
Therefore, this can be used for connecting directly to lights but even powerplugs could be supported.  If more power is required, relays can be added.
[The RS485 gateway] (https://www.waveshare.com/rs485-to-wifi-eth.htm) Was selected due to the very flexible DC range for power (5V and higher), the Wifi/UTP connection and the support for MQTT.
However during the implementation, I noticed that the MQTT is not supporting the control per channel and the LAN/Wifi set-up is very confusing.  It's very powerful that you can connect the gateway to UTP and use the Wifi as an access point to your network, but it is not possible to only use the UTP as you can't disable the Wifi. :(

### The challenges during the activation
* The manuals of Waveshare are very confusing and the tools delivered via Wavecom (SSCOM and Vircom) are just not working.  None of the Waveshare tools was able to detect the Waveshare devices so I lost hours assuming that the modbus was not working. The only good tool is a commercial tool called [Modbus Poll] (https://www.modbustools.com/modbus_poll.html).  However it is limited in time, you can easily test the 16-channel relay.
* The userinterface of the Wavecom gateway is a nightmare and you don't have any diagnostictools from the interface.  
* The modbus plugin in Home Assistant can only be integrated via direct configuration in configuration.yaml.  However, the yaml of HA are very sensitive to typo's and event indents of text.  Every change requires a full reboot of the Home Assistant server.  Additional problem is that Home Assistent made a lot of syntax changes in the config files lately so online resources are typical referring to none working configs

### How to configure (assume modbus is correctly connected between gataway and 16-channel relay)
* The Waveshare gateway
In a default configuration, the Waveshare gateway is an AP with a webinterface.  (http://10.10.100.254, admin/admin).  In that interface, it's important to set the baud (serial speed of RS485 bus is fixed for all devices), to choose to use the UTP or Wifi (STA) to integrate into your network and to change the port settings in the application screen to 502.  Also, don't forget the set the mode of operations to TCP RTU to RS485 RTU.  Note that RTU is the used format of communication.  You have RTU and ASCII but only RTU is supported on Waveshare.
See photo's:

![alt text](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/mode_configuration.png)

Choose the TCP to RS485 RTU.  Transparent will not work.

![alt text](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/serialSettings.png)

Make sure you set to default 9600 8N1

![alt text](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/ModBusRTU_port502.png)

In application settings, use the port 502 as this is a market standard

![alt text](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/LANaccess.png)

If you want to make sure that you gateway has a fixed IP, go into the AP settings and update the IP address (I disabled DHCP server assuming as my network has a DHCP server)

![alt text](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/routerofbridge.png)

Very confusing setting but here, you will see that the gateway can be bridge (z) or gateway (n).  In the bridge mode, the wifi AP will become a hotspot with access to your network.  In case of gateway, you get an AP with his own IP range that will route to Internet or your network.  I didn't want any security tunnel to 3th party networks, so took bridge.

* The Waveshare 16 channel relay
You can't configure anything on this relay.  You could try to make changes to the relay with the AT commands to for example change the slave "ID" on the RS485 bus.  Default, this is "1" and in Home Assistant, you can see that I didn't had to put this ID as default is also "1".   I didn't try what happens if other devices are added to the RS485 bus.  

* Test if you can use the 16 channel relay
This is only possible from Windows with the [Modbus Poll] software.  With this software, you can set the gateway details (IP and TCP port) and the number of channels.   Put the mode on "1" or read and you can see the status of the relays.  You can even change and if all is fine, you here a nice click on the relays.

![alt text](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/debug.png)

![alt text](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/debug2.png)

* The Nightmare -- sorry challenge with Home Assistant
The only way I was able to get this working was thanks to a night of AI with Google Gemini and ChatGPT.  Google Gemini helpt a lot on the configuration.yaml but could solve the issue with the communication issues with modbus gateway.  However, I knew that there was a solution as I was able to read and write to the 16 channel relay with Python. (code can be found on WaveShare portal with name Modbus_RTU_Relay_16CH_Code.zip)  Note that the code is build for serial RS485 communication, but you can easily convert this to TCP RTU. [see here](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/writecoils.py)

The main problem with Home Assistant is that the default modbus module is not able to communicate correctly with the gateway.  You could read but not write.  Thanks to ChatGPT, I was able to find a way to directly test the connections without going through the UI.  The trick is that you can execute actions directly from the developer tools and actions.  Using the actions, I could read and write the relays.

![alt text](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/DebugHA.png)

After checking, the only way I was able to use the modbus integration is by using only the config of modbus with dummy switches and add a template of switches. [see here](https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/configuration.yaml)
Note that it took me hours to find the working configuration format as ChatGPT and Google Gemini are still working with the outdated formatting.  After a lot of trials, I was able to get it working.




