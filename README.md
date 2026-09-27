# Saeco GranAroma Deluxe: switch on with Wake-on-LAN

The Wi-Fi-connected Saeco GranAroma Deluxe (SM6680 / SM6685) switches on from
standby when it receives a standard Wake-on-LAN magic packet. It goes back to
standby by itself.

It will likely work with other Wi-Fi-enabled Saeco machines of the same
generation too, such as the Xelsis Deluxe and Xelsis Suprema. Those are
untested. If you try one, please open an issue with the result.

## Usage

Find the machine's MAC address in your router.

```bash
wakeonlan 34:6f:24:xx:xx:xx
```

## Home Assistant

### 1. Enable Wake on LAN

Add this line to `configuration.yaml` and restart Home Assistant:

```yaml
wake_on_lan:
```

This enables the [Wake on LAN](https://www.home-assistant.io/integrations/wake_on_lan/)
integration, which gives you the `wake_on_lan.send_magic_packet` action.

### 2. Create the script

Go to **Settings → Automations & scenes → Scripts → Add script**. Open the
⋮ menu, choose **Edit in YAML**, paste this and put your machine's MAC in:

```yaml
alias: Coffee machine on
icon: mdi:coffee-maker
mode: single
sequence:
  - repeat:
      count: 1
      sequence:
        - action: wake_on_lan.send_magic_packet
          data:
            mac: "34:6f:24:xx:xx:xx"
        - delay:
            seconds: 2
```

One packet is usually enough. If your Wi-Fi is unreliable and the machine
sometimes doesn't wake, raise `count` to 2 or 3. A Wi-Fi broadcast can get
lost, and the delay spaces the retries out. Extra packets don't affect a
machine that is already on. The same script is in
[`home-assistant/script.yaml`](home-assistant/script.yaml) in `scripts.yaml`
format.

Run the script once from its ⋮ menu to test it. Let the machine sit in
standby for a minute first: a packet sent right after switching it off may be
ignored.

### 3. Add a dashboard button

Edit your dashboard, add a **Button** card and choose the script. Tapping the
button runs it. The same card in YAML:

```yaml
type: button
entity: script.coffee_machine_on
name: Coffee
icon: mdi:coffee-maker
tap_action:
  action: perform-action
  perform_action: script.turn_on
  target:
    entity_id: script.coffee_machine_on
```

Home Assistant can't tell whether the machine is on, so use a button that
runs the script, not an on/off switch.

### Troubleshooting

- **Nothing happens:** check the MAC address, and that Home Assistant is on
  the same network as the machine. A packet sent from a different VLAN or
  subnet won't reach it.
- **Home Assistant in Docker:** the container needs `network_mode: host`, or
  the broadcast never leaves the container. Home Assistant OS needs nothing
  extra.

## License

MIT
