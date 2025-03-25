# 🔌 Shelly Gen2/Gen3 Auto-Discovery for Home Assistant (MQTT)

This setup enables **automatic MQTT discovery** of Shelly Gen2/Gen3 devices in Home Assistant using the `shellies_discovery_gen2.py` Python script and a simple automation that listens for discovery messages published to `shellies_discovery/rpc`.

---

## ✅ What It Does

- Listens for MQTT messages from Shelly devices (Gen2/Gen3) on `shellies_discovery/rpc`
- Parses each discovery message
- Automatically registers the device via MQTT Discovery
- No need to manually configure each Shelly device

---

## 📁 Files Used

### 1. `python_scripts/shellies_discovery_gen2.py`

> [Source repo here](https://github.com/bieniu/ha-shellies-discovery-gen2)

This Python script processes incoming MQTT discovery payloads from Shelly devices and publishes them to the correct `homeassistant/.../config` topics for seamless integration.

---

### 2. Automation: `Shelly Gen2/Gen3 MQTT Discovery`

```yaml
alias: Shelly Gen2/Gen3 MQTT Discovery
description: Auto-discovers Shelly Gen2/Gen3 devices using shellies_discovery_gen2.py
mode: single
trigger:
  - platform: mqtt
    topic: shellies_discovery/rpc
condition: []
action:
  - choose:
      - conditions:
          - condition: template
            value_template: >
              {% set p = trigger.payload | default('') %}
              {{ p is string and p.startswith('{') and '"src"' in p }}
        sequence:
          - variables:
              payload: "{{ trigger.payload | from_json }}"
          - service: python_script.shellies_discovery_gen2
            data:
              id: "{{ payload.src }}"
              type: "{{ payload.src.split('-')[0] }}"
              fw_ver: >
                {% if 'result' in payload and 'sys' in payload.result and 'config' in payload.result.sys and 'fw_id' in payload.result.sys.config %}
                  {{ payload.result.sys.config.fw_id }}
                {% else %}
                  unknown
                {% endif %}
              mac: >
                {% if 'result' in payload and 'sys' in payload.result and 'config' in payload.result.sys and 'mac' in payload.result.sys.config %}
                  {{ payload.result.sys.config.mac }}
                {% else %}
                  unknown
                {% endif %}
              device_ids:
                - shelly1pmg3-3030f9eb7774
                - shelly1pmg3-3030f9e94028
                - shelly1pmg3-3030f9e96fa4
                - shelly1pmg3-3030f9eb4a34
                - shelly1pmg3-3030f9eb68cc
                - shelly1pmg3-3030f9e86648
                - shelly1pmg3-3030f9e965d0
                - shelly1pmg3-dcda0cb12584
    default: []
