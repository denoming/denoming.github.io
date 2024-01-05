---
title: "Configure Arduino IDE to support ESP-01"
date: 2021-07-17
categories: [Engineering,ESP]
tags: [esp01]
---

1. In your Arduino IDE, go to `File` => `Preferences`;
2. Enter http://arduino.esp8266.com/stable/package_esp8266com_index.json into the `Additional Boards Manager URLs` field. Then, click the `OK` button;
Note: URLs can be separated by comma.
3. Open the board manager by `Tools` => `Board: ...` => `Board Manager...`;
4. Search for `ESP8266` and press install button for the `ESP8266 by ESP8266 Community`;
5. That's it.

After that, a lot of examples for ESP8266 become available.
