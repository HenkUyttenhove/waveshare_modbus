# Home Assistent with WaveShare 16 channel relay with Modbus RTU

### Objective of this project
Domotica is here to stay and [Home Assistant] (https://www.home-assistant.io/) has further lowered the need for technical specialists.
There are different vendors on the IoT market delivering Wifi/Zigbee based solutions and I also have already deployed a Sonoff network.
However, the need for wireless signals and the dependency on the Sonoff online gateway to support integration with platforms as Google Home and Home Assistent is a limitation.

While I was surfing on the Internet, I noticed the very cheap Waveshare relay systems.  These however are running on modbus TCP or modbus RS485.  
The modbus TCP may sound as the easiest platform but because I'm also looking at integration of solar panels and heat pump, I went for the RS485 to learn more about this.
AS RS485 is a special 2-wire protocol (A+B and ground), you need an additional gateway.  For this gateway, you can take the older Waveshare RS485 LAN gateway, but I took the newer Waveshare RS485 Wifi-LAN device.

(https://github.com/HenkUyttenhove/waveshare_modbus/blob/main/overview.jpg)




