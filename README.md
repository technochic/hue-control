# Controlling the Philips Hue Hub

Philips [Hue lighting system](http://www2.meethue.com/en-us/) allows you to control the lighting in your home from a mobile app, or from any application you write yourself that can connect to the internet. The center of the system is the Hue hub, an Ethernet-connected device that communicates with compatible lamps through the [ZigBee HA](http://www.zigbee.org/zigbee-for-developers/applicationstandards/zigbeehomeautomation/) radio protocol. Philips makes a range of Hue-compatible lamps, and the [GE Link lamps](http://gelinkbulbs.com/) also work with the Hue system. Each Hue hub is its own HTTP web server, and can be controlled using the [Hue REST API](http://www.developers.meethue.com/). There are libraries to control the Hue available in many programming languages. The tutorials here are all in client-side JavaScript using [P5.js](http://p5js.org/), or server-side using [node.js](https://nodejs.org/), or micrcontroller-based for [Arduino](http://www.arduino.cc).

## Useful Tools

* To get started programming Hue apps, you'll need access to a Hue hub. You'll want a [Hue account](https://my.meethue.com/en-us/) too. The developer accounts are free. Keep the [Hue Developers Site](http://www.developers.meethue.com/) link handy.

* The Hue app for [Android](https://play.google.com/store/apps/details?id=com.philips.lighting.hue&hl=en) or [iOS](https://itunes.apple.com/us/app/philips-hue/id557206189?mt=8) is helpful when developing, because it works when your project doesn't yet. The [Hue Essentials app](https://www.hueessentials.com/) is a pretty helpful alternative also.

* Every Hue has a debug interface, available at `http://$ADDR/debug/clip.html` Replace `$ADDR` with your hub's IP address. When you're developing, you can use this to send API commands to the hub to test things out.

* Peter Murray's [node-hue-api library](https://github.com/peter-murray/node-hue-api) for node.js is the best of the various node.js JavaScript libraries I've tested for controlling the Hue.

* For controlling the Hue from a browser client, [p5.js](https://p5js.org) does a good job, as it's got a simple [http request API](https://p5js.org/reference/#/p5/httpDo). 
* The [ArduinoHTTPClient library](https://github.com/arduino-libraries/ArduinoHttpClient) and the [Arduino_JSON library](https://github.com/arduino-libraries/Arduino_JSON) are useful if you're using any of the Arduino WiFi-enabled boards to connect to your Hue hub. (Note: there's another JSON library by the same name with no underscore. That one is not the one used here).
* The command line tool [curl](https://curl.haxx.se/docs/httpscripting.html) is really helpful to test HTTP requests to your hub. Curl's not available in the Windows command interface, but you can get it through the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) in Windows 10, or through [cygwin](https://www.cygwin.com/), an application that provides a linux shell for Windows.  

Other than these, you'll need to know some HTML and JavaScript, and a text editor, a command line interface, and a browser to try the examples on this site.

## Connecting To Your Hub

Your app will need to be identified to your hub using a unique username. Follow the developer site's [Getting Started instructions](http://www.developers.meethue.com/documentation/getting-started) to add a new app/username to your hub. Here they are in brief:

Open a browser and connect to the hub's debug interface:

    http://$ADDR/debug/clip.html

![Hue debug clip interface](images/debug_clip.png)

In the URL field, enter:

    /api

And in the Message Body, enter

    {"devicetype":"my app"}

You can use any value you want for devicetype. Older versions of the Hue hub firmware let you enter your own username, but newer versions auto-generate the username.

Press the link button on the hub and click POST. You should get a response like this:

    [
    	{
    		"success": {
    			"username": "newusername"
    		}
    	}
    ]

Now you're ready to write code for your hub. Regardless of what environment you're programming in, you'll use the username you established here.

_Note:_ recent versions of the Chrome browser have disabled the debug interface. If you can't get to the debug interface, you can do everything it can do through  a command line interface using [curl](#connecting-through-curl).

## Other Features of the Debug Clip Interface

The debug clip interface can be used to send any API command to your hub. The general query string is as follows:

    /api/$HUE_USER/command

Generally, querying the state of the device is done using GET commands, and changing them is done using PUT. For example, to query the state of all your lights, enter the following in the URL field:

    /api/$HUE_USER/lights

 And click GET. To query the configuration, enter:

     /api/$HUE_USER/config

And click GET.  To turn on light 1, enter the following in the URL field:

    /api/$HUE_USER/lights/1/state

And enter the following in the Message Body field:

    {"on":true}

And click PUT. The light should come on, and the hub should reply:

	[
		{
			"success": {
				"/lights/1/state/on": true
			}
		}
	]

For more on the Hue API, see the [Hue Getting Started guide](http://www.developers.meethue.com/documentation/getting-started),  the [Hue API Core Concepts](http://www.developers.meethue.com/documentation/core-concepts), and the full [Hue API documentation](http://www.developers.meethue.com/philips-hue-api) (you'll need to log in for this).

## Connecting Through Curl

You can connect to your Hue hub using a command line interface as well, with the curl program, as mentioned above. For example, here's how you'd create a new user using curl:

Enter the following on the command line, replacing $ADDR with your hub's IP address:

    $ curl -X POST -d '{"devicetype":"my app"}' http://$ADDR/api

You can use any value you want for devicetype. Press the link button on the hub BEFORE YOU HIT ENTER on this command, then hit enter. You should get a response like this:

    [
    	{
    		"success": {
    			"username": "newusername"
    		}
    	}
    ]

Now you're ready to write code for your hub. Regardless of what environment you're programming in, you can use the username you established here. Most apps establish a unique username for the app.

Assuming you have a Hue user set up on the hub, and you've got a lamp already connected to the hub, here's a quick set of commands to test from the command line, using curl. The first couple of commands establish your username and hub address in command line interface environment variables so you don't have to retype them.
````
$ export HUE_USER='fill-in-your-hue-username-given-to-you-by-the-hub' 
$ export ADDR='ip.address.of.hub'
$ curl http://$ADDR/api/$HUE_USER/lights/   
# this should return the list of available lights, like so:

{"1":{"state":{"on":true,"bri":254,"hue":14314,"sat":172,"effect":"none","xy":[0.4791,0.4139],"ct":405,"alert":"none","colormode":"ct","reachable":true},"type":"Extended color light","name":"Hue color light 1","modelid":"LCT001","manufacturername":"Philips","uniqueid":"00:17:88:01:00:ff:9a:28-0b","swversion":"5.127.1.26581"}}

$ curl -X PUT -d '{"on":true}' http://$ADDR/api/$HUE_USER/lights/1/state
# The light should now be on
$ curl -X PUT -d '{"on":false}' http://$ADDR/api/$HUE_USER/lights/1/state 
# The light should now be off
````

## Finding Your Hub's IP address

When you've added your hub to your network, you should be able to use the Hue app or the Hue Essentials app to get the IP address. But on a complex network like a school network, that may not work. Your mobile device and your Hue hub have to be on the same local network for this to work. For example, if your WiFi network is not the same local net as your wired Ethernet network (where the hub lives), you may not be able to get the address. But if you can get the hub's MAC address, then you can search for it on your network. Here's how

Every hub has a unique You can find the MAC address on the bottom of your hub. It's a six-byte number in hexadecimal notation like so:

    00:17:88:0B:14:48

Some hubs will only show the last three bytes. For example, the hub above might show just 0B1448. With older Hue hubs, the first three bytes will always be 00:17:88. With newer ones, you might also see EC:B5:FA instead.

To look for your hub on your network, make sure you have the first trhee digits of the local network, and that you can access it, then open a command line interface and type:

    $ ping -c 5 xxx.xxx.xxx.255 

Where xxx.xxx.xxx are the first three numbers of your network. For example, on a network whose router is 172.16.130.1, you'd enter 172.16.130.255. Sometimes large institutions will use two different subnets for wired vs. wireless networks, but they will still be on the same larger local network. 

You'll get a  list of responses, as devices on the network respond to your ping requests.  When it's done, type:   

    $ arp -a


You'll get a list of all the devices on the same network that your computer can see. Look for the one that matches the MAC address of your hub, and the IP address next to it will be your hub's IP address. The last three bytes of your MAC address from the label on the bottom. The first three will likely be either 00:17:88 or EC:B5:FA, as explained above. here's a typical example:

    $ arp -a
	? (192.168.0.1) at ac:b7:16:61:e3:77 on en0 ifscope [ethernet]
	? (192.168.0.3) at 00:17:88:0B:14:48 on en0 ifscope [ethernet]
	? (192.168.0.255) at ff:ff:ff:ff:ff:ff on en0 ifscope [ethernet]

In this case, the hue's IP address is 192.168.0.3. 

## Search for New Lamps

You can search for new lamps on the hub using the regular mobile Hue app. You can also do it from the debug clip interface using a POST request on the following URL:

    /api/$HUE_USER/lights/

In curl, that's:

    $ curl -X POST http://$ADDR/api/$HUE_USER/lights

Fill in your hub's address for $ADDR and your hue username for $HUE_USER.  You should get a reply like this:

    [[ { "success": { "/lights": "Searching for new devices" } }]

After 90 seconds, you can scan for new lamps that were added like so:

    /api/$HUE_USER/lights/new

In curl:
    
    $ curl -X GET http://$ADDR/api/$HUE_USER/lights/new

This will list only the new lamps added after a scan for new lamps. 


## Capturing a Lamp From Another Hub

If you're trying to add a lamp that was previously connected to a different hub, you'll need to use a different approach. Place the lamp close to the hub with which you want to control it (closer than 30cm, or 1 ft). Turn off all other lamps connected to the hub, or make sure they're much further away than the one you want.

Send the following the debug clip interface using a PUT request:

    /api/$HUE_USER/config/

In the body of your request put:

    {"touchlink": true}

In curl that's:

    $ curl -X PUT -d '{"touchlink": true}' http://$ADDR/api/$HUE_USER/config


The lamp should blink a few times, and the server will respond with a success message. You can now add the lamp using the find new lamps request described above.

## Deleting a Lamp from a Hub

You can delete a lamp from a hub from the debug clip interface using a DELETE request on the following URL:

    /api/$HUE_USER/lights/1

In curl:

    $ curl -X DELETE http://$ADDR/api/$HUE_USER/lights/1 

Replace 1 with the number of the light you wish to delete.

## Getting the State of All Lights

To get the status of all connected lights, send the following from the debug clip interface using a GET request:

    /api/$HUE_USER/lights/

In curl, that's:

    $ curl http://$ADDR/api/$HUE_USER/lights/   

This should return the list of available lights, like so:
 
    {"1":{"state":{"on":true,"bri":254,"hue":14314,"sat":172,"effect":"none","xy":[0.4791,0.4139],"ct":405,"alert":"none","colormode":"ct","reachable":true},"type":"Extended color light","name":"Hue color light 1","modelid":"LCT001","manufacturername":"Philips","uniqueid":"00:17:88:01:00:ff:9a:28-0b","swversion":"5.127.1.26581"}}
 
## Turning a Light on

To turn a light on you need to know which number it is. Then you change its state like so:

Send the following from the debug clip interface using a PUT request:

    /api/$HUE_USER/lights/4/state

In the body of your request put:

    {"on": true}

In curl, that's:

    $ curl -X PUT -d '{"on":true}' http://$ADDR/api/$HUE_USER/lights/4/state

To turn it off, change the body of the request to 

    {"on": false}

You can change any of the properties of a light's state this way. Take a look, for example, at light 1 from the [Getting the State of All Lights](#getting-the-state-of-all-lights) section above:

    {"1":{"state":{"on":true,"bri":254,"hue":14314,"sat":172,"effect":"none","xy":[0.4791,0.4139],"ct":405,"alert":"none","colormode":"ct","reachable":true},

As long as the `reachable` property is true, meaning that the hub tried to reach the lamp and got a response, you can change any of the other properties. This is a color lamp, and has three modes, hs (for hue, saturation), ct (for color temp), and xy (for x and y dimensions in the CIE1931 color space). If you change either the hue or saturation, the lamp's colormode changes to hs, and if you change the color temperature, the colormode changes to ct. If you send xy values, the colormode changes to xy mode.

* bri (brightness): 0-254
* hue: 0-65535, through the colorwheel from red to red
* sat (saturation): 0-254
* effect: "none" or "colorloop" see [Developer API docs](https://developers.meethue.com/develop/hue-api/lights-api/)
* xy (x and y position in [CIE1031 colorspace](https://medium.com/hipster-color-science/a-beginners-guide-to-colorimetry-401f1830b65a)): 2 floats, 0.0 to 1.0 each
* ct (color temperature): in [mired](https://en.wikipedia.org/wiki/Mired), 153 - 500
* alert - see [Developer API docs](https://developers.meethue.com/develop/hue-api/lights-api/)

Different lights will have different properties in their state variable that you can change. The p5.js sketch will scan all the properties of each lamp on the 

## Adding GE Link Lamps to the Hue

You can add [GE Link lamps](http://gelinkbulbs.com/) to the Philips Hue as well, since they communicate using the same ZigBee protocol. To do this, turn the Link lamp on and connect it just as you would a regular Hue lamp. It will show up as _Dimmable Lamp 1_. You'll only be able to control brightness.

### Resetting GE Link Lamps

If you can't connect a GE Link lamp to your hub, you might need to reset the lamp. To do this, turn the lamp on for three seconds, then off for three seconds. Repeat this five times until the lamp dims down and blinks once. The lamp is now reset. Try to connect again.

For more on the GE Link lamps, see their [install guide](http://gelinkbulbs.com/downloads/GELinkInstallGuide.pdf).

## Arduino Control of Hue Lights

The MKR1000, MKR1010, and Nano 33 IoT Arduino models can control the Hue hub via HTTP requests as well. There are some [Arduino Hue examples](ArduinoExamples), with notes, in this repository as well.

## Further Reading

* [Client-Side Programming of the Hue Hub](client-programming.md)
* [Server-side Programming of the Hue Hub](server-programming.md) in node.js
