# Bambu Labs Proxy

This is a proxy for the Bambu Labs printer MQTT broker. It uses Mosquitto to multiplex many clients to a single printer without hitting the connection limits. If you do not have a BBL printer, DO NOT BUY ONE. Get something else. Seriously. I'm not kidding. If you really want one still, I'll sell you mine.

If you are already unfortunately stuck with one, this is a pretty big quality of life improvement.

This configuration creates a dedicated relay that can multiplex hundreds of connections, not just 4. Printer clients (xtouch or octoprint or whatever) connect to the relay directly. Optionally, you can even bridge the dedicated relay to your main MQTT broker. (I did.)

    (Optional: Existing MQTT broker)
            || - Optional
    (THIS: Bambu Relay Mosquitto Broker)
            || - printer.conf
    (Bambu Labs Printer)

It is presented as a Flux Helm installation but it shouldn't be any harder to decode than a docker-compose would be. Let me know if there are questions.

## What it does not do

- Anything non-MQTT. Tools that go beyond standard MQTT require a direct connection.
  - No chamber camera (I have a proxy for that, if there is interest.)
  - No file management (yet..)
  - Pinging or portscanning or otherwise doing something strange to determine if there is a printer

## What Works

Everything that uses only MQTT should be fine. This configuration relays the requests to the printer and reports back, while ignoring everything else. I use this with the following projects:

- [WolfWithSword](https://github.com/WolfwithSword/Bambu-HomeAssistant-Flows)'s Node Red flows
  - Requires hacking on the "printer alive" test. Author is [unwilling](https://github.com/WolfwithSword/Bambu-HomeAssistant-Flows/issues/15#issuecomment-2466936542) to put an input hook in for environments where ping is inappropriate. but the change can be done by hand on every update. (Instead of ping, it should check `$SYS/broker/connection/printer/state`.)
- [xTouch](https://github.com/xperiments-in/xtouch) esp32 control panel
  - New versions are cloud-only, so it requires an [older version](https://github.com/xperiments-in/xtouch/issues/130)
- [Octoprint BBL](https://github.com/jneilliii/OctoPrint-BambuPrinter/) sometimes works great
  - There is some dependency on ftp in there that messes with it. Might be an Octoprint architectural problem.
  - Lots of errors in the logs about the chamber images.
  - When it misbehaves, I point it straight to the printer and it works as well as it ever does. (Not great, but that is not the author's fault.) Since the relay only uses one connection, and the slicer uses one, there is usually one available for Octoprint.
- [Home Assistant](https://github.com/greghesp/ha-bambulab) via HACS.
  - Seems to be fine, even combined with the NR flows.
  - Had to rename a some devices to prevent conflicts.
  - Shows much more accurate information than Octoprint, such as ETA and current layer.
