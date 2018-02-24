# SmartThings-and-Squeezebox-DLNA-Player
Extends SmartThingsUle/DLNA-PLAYER to work with multiple Squeezeboxs

If you use it please consider donating to my favourite charity: https://www.nowdonate.com/checkout/pv0j03m4s1o1x60o6bh2

The motivation for this project was to get announcements form Squeezebox Players.

This project allows the detection of Logitech Squeezeboxs that are running from a LMS with the plugin UPnP/DLNA Media Interface by Andy Grundman by extending the DLNA-PLAYER Smart App and  Device Handler https://github.com/SmartThingsUle/DLNA-PLAYER.

The project has turned out to be a lot bigger than I thought.
The issues That had to be overcome include
1. Allowing multiple devices for DLNA renderers on a single IP address.
2. Setting the correct subscribe path.
3. Returning replies to the correct player.

Detected Squeezebox media renderers devices are made using the uuid not the IP address and port. The IP address and port are now device settings.
The subscribe path is now modified, based on the detection of a 'http:' leader, to include the appropriate port and renderer identification (the SB Player MAC)
I use a cheat to elicit a 'Set-Cookie' response from the SBS for the each particular device so that reply messages can be associated with the correct renderer. This is not fool proof but works for the main use cases. All the replies from the MAC address on which the SBS is running are received by a new device 'LAN Handler' that forwards it to the Media Render (Connect) Smart App which finds the Set-Cookie and forwards it on to the correct Media player Device (child device).
There one specific line 242 of the smart App that returns an error under certain conditions, but it doesn't appear to prevent the sending of the event. Help greatly received
' d.subscribeResp(response) // This line gives an error but sends anyway'

I have included a simpler version of the LAN Handler Device as my LAN Handler manages several other devices running on the same RasPi as my SBS.
The LAN Handler has to be selected in the Smart App Settings.
Also included is the modified Smart App and Device Handler.

I have tested it with 5 Squeezebox players and all works pretty well. I have not tested with multiple players from different manufacturers, so there may need to be a few extra gates in some of the functions.
I have tested with 'Big Talker' and this works well to allow announcements, however the UPnP interface on the SBS appears to have some separation from the various native SB player interfaces which hamper the resumption of playback.
I have done some limited testing with Bubble UPnP Server (Android App) and this provides a more seamless integration. I expect that if you usually play your media on a SB player using UPnP you will not see the problem above.
For details on how to set this up see SmartThingsUle repository.
I found that the 'Delay before msg' and 'Delay between actions' in the Squeeze box devices need to be set to 1 second.
Don't forget to install the LAN Handler with a DNI set to the MAC of the device you are running your SBS on. Also set this LAN Handler in the smart App settings.
The result is by no means polished, and I'm not much more than a tinkerer when it comes to coding so there are probably better ways of implementing some things, but we are all here to learn and progress.
