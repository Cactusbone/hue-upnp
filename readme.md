Edit the properties in the beginning of the python-server and start with python hueUpnp.py

Notes from falk0069:

I don't own a Phillips hue but I wanted an emulator so I could test it against my Harmony remote which supports them.  I found a number of emulators but none of them worked with the Harmony remote.  Sagen's python code was easiest to modify and do what I needed.  I've never coded in python before so this was a good experience.

Some pitfalls to mention:
1) The UDP response needs to be bound to port 1900 (like the request) and sent to the same port that the client used (for Harmony)

2) Harmony sends three ST requests at a time:
ST: ssdp:all
ST: urn:schemas-upnp-org:device:basic:1
ST: urn:schemas-udap:service:smartText:1

3) The Harmony remote will only progress if you reply back with the same ST.  All other examples online use this ST instead:
ST: upnp:rootdevice

4) I got hung up on the smartText one for a while.  It turns out this has something to do with LG TVs.  I have no idea why it is using it.

5) The HTTP PORT used can be anything and the Harmony remote will use it.  However, if you use the standard Phillip Hue Android (and iOS?), you need to use port 80.

6) I added in a EXTERNALPROG call, so anyone can toss in their own commands.  I use this so I can remotely call WakeOnLan using my remote.  I also use it to call a Wemo On-Off script.  For whatever reason Logitech doesn't support Wemos (yet).

****************************************************************************
Aug 2015:
The Logitech firmware changed a bit on how it accepts communications.  For some reason it likes to send HTTP requests in two separate packets.  This is not new.  For example:

Packet 1:
GET /api/lights HTTP/1.1

Packet 2:
connection: close, TE
content-length: 0
user-agent: LuaSocket 2.0.2
te: trailers
content-type: application/json
host: 192.168.1.10

If we only acknowledge packet 1 and continue to wait for the rest of the data, it freaks out and terminates the connections. It seems that it wants to see the reponse *before* it has finished making the request.  I'm quite baffled on this.  Since the data in the second packet isn't required to respond, I now send the payload right away.  Packet 2 still gets sent, but it causes a RST packet to be generated since it is arriving after the connection is now done.  Oh well, it least it is working again.
