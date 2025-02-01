# Bambu Labs Proxy

This is a proxy for the Bambu Labs printer MQTT broker. It uses Mosquitto to multiplex many clients to a single printer without hitting the connection limits. 

If you do not already have a BBL printer, **DO NOT BUY ONE**. Get something else. Seriously. I'm not kidding. If you really want one still, I'll sell you mine.

If you are already unfortunately stuck with one, this is a pretty big quality of life improvement.

This configuration creates a dedicated relay that can multiplex hundreds of connections, not just 4. Printer clients (xtouch or octoprint or whatever) connect to the relay directly. Optionally, you can even bridge the dedicated relay to your main MQTT broker. (I did.)

    (Optional: Existing MQTT broker)
            || - Optional
    (THIS: Bambu Relay Mosquitto Broker)
            || - printer.conf
    (Bambu Labs Printer)

Most of the testing is non-existant, and the testing that I have done is largely for the Helm method, but there is a docker compose example that worked for me at least once. If it breaks, open an issue. If you want a different deployment example, open a pull request. (It doesn't have to be perfect, but give it a try anyway. I'm unlikely to do it for you, especially if you are asking me to learn your tool of choice.)

## What Doesn't Work (yet)

- Anything non-MQTT. Tools that go beyond standard MQTT require a direct connection. For example:
  - No chamber camera (I have a proxy for that, just have to integrate it here)
  - No file management (proftpd might be able to solve this)
  - Pinging or portscanning or otherwise doing something strange to determine if there is a printer

## What Does Work

Everything that uses only MQTT should be fine. This configuration relays the requests to the printer and reports back, while ignoring everything else. I use this with the following projects:

- [WolfWithSword](https://github.com/WolfwithSword/Bambu-HomeAssistant-Flows)'s Node Red flows
  - Requires hacking on the "printer alive" test. Author is [unwilling](https://github.com/WolfwithSword/Bambu-HomeAssistant-Flows/issues/15#issuecomment-2466936542) to put an input hook in for environments where ping is inappropriate. but the change can be done by hand on every update. (Instead of ping, this relay puts the status under `$SYS/broker/connection/printer/state`.)
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

## Getting Started

### Advanced Users
Check `examples/configs` for the raw configs. Toss the printer's certificate into `blcert.pem`, create the users file with `mosquitto_passwd` and update the `mosquitto.conf` with your printer's values. Start using the example of your choice (helm, docker compose, jails, etc.)

### Everyone Else
You probably want docker compose. Edit or create the files under `examples/config` and `examples/docker/docker-compose.yaml`. Review everything flagged with `##`.

Once the configuration is ready, start it with `docker compose up --remove-orphans`.
