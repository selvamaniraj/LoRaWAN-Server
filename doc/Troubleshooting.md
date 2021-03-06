# Troubleshooting Instructions

Overview of recent errors and warnings is provided in the server [Event List](Events.md).
Details can be found in the server logs.

The server logs are stored in `lorawan-server/log`. By default three log files are
provided: `debug.log`, `error.log` and `crash.log`. The log messages contain date
and time, severity (debug, info, notice, warning, error), process ID and a description.

For example:
```
2017-03-05 02:14:57.148 [info] <0.361.0> Gateway <<1,39,235,255,255,176,23,196>> at {192,168,0,1}:47002
```

Binary identifiers are often shown in the Erlang binary notation, enclosed in `<< >>`. This
is simply a list of bytes in decimal notation.
For example, `<<1,39,235,255,255,176,23,196>>` means `01-27-EB-FF-FF-B0-17-C4`.

By default no messages are shown on the console. To see the debug messages you need to
open the server configuration in `lorawan-server/releases/<VERSION>/sys.config`,
uncomment (remove the `%` character) from the `{lager_console_backend, debug}` line
and restart the server.

## Common Errors

### No message in log file

The log shall always contain at least an [info] record `Gateway <<MAC>> at {IP}:PORT`.
If this is not listed, the gateway is down or the network connectivity is broken.

This may be due to:
 * Gateway misconfiguration. Check the gateway configuration includes a correct
   server IP and port (by default 1680).
 * Firewall misconfiguration. Check the server firewall does not block the port 1680.
   See [Installation Instructions](Installation.md) for configuration guidelines.

### Join request received, but no further frames

This is usually because the downlink frame including the Join response was not sent or
received by the device.

This may be because:
 * Gateway error. Check for any error in your gateway log file.
 * The device did not listen on the channel (frequency) used by the server. Verify
   your device listens in the RX1 window.

### unknown_mac

The gateway *MAC* is not correctly configured.
Open the [Web Administration](Administration.md), go to the *Infrastructure-Gateways*
list and check the gateway *MAC* is correctly listed.

### unknown_deveui

The over-the-air activated (OTAA) device *DevEUI* is not correctly configured.
Open the [Web Administration](Administration.md), go to the *Devices* list
and check the device EUI is correctly listed.

### unknown_devaddr

The device *DevAddr* activated-by-personalization (ABP) is not correctly configured.
Open the [Web Administration](Administration.md), go to the *Nodes* list
and check the device address is correctly listed.

If you see strange *DevAddr* numbers from devices that you don't know, there
may be a second network near you. You can add devices from this network into
the list of [Ignored Nodes](Infrastructure.md).

### bad_mic

The Message Integrity Check (MIC) of a received frame has failed.
 * If this appeared only once this was due to a transmission error.
 * If this happens periodically with the same ID, the device is not correctly
   configured.
   * When this is related to a 8-bytes long *DevEUI*, open the
     [Web Administration](Administration.md),
     go to the *Devices* list and check the *AppKey*.
   * When this is related to a 4-bytes long *DevAddr*, go to the *Nodes* list
     and check the *NwkSKey*.

### fcnt_gap_too_large

The device sent an unexpected frame counter *FCnt*. This may be because:
 * The device is activated-by-personalization (ABP) and it did reset recently.
 * The device was unreachable for a very long time.

If this is an exceptional case, go to the *Nodes* list and manually update the
*FCnt Up* to the *FCnt* number.

To allow ABP devices to freely reset set the *FCnt Check* to *Reset on zero*,
but please note this weakens LoRaWAN security a bit.
It is recommended to use over-the-air-activation (OTAA) instead.

### prerequisite_failed

This is reported when the lorawan-server is started with older Erlang/OTP. At
least 19.0 (or later) is required.
