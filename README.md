# esp8266-sprinkler-controller
ESPHome build of sprinkler controller using 4 channel relay





# BEGIN SUBSTITUTIONS
substitutions:
  name: "esp8266-esp01-01"
  friendly_name: "BackYard Sprinklers"
  # Allows ESP device to be automatically lined to an 'Area' in Home Assistant. Typically used for areas such as 'Lounge Room', 'Kitchen' etc
  device_description: "esp8266-esp01-1M with 4 channel uart relay"
  sensor_update_interval: "10s"
  # Set timezone of the smart plug. Useful if the plug is in a location different to the HA server. Can be entered in unix Country/Area format (i.e. "Australia/Sydney")
  timezone: "America/Regina"
  # Set the duration between the sntp service polling ntp.org servers for an update
  sntp_update_interval: "6h"
  # Network time servers for your region, enter from lowest to highest priority. To use local servers update as per zones or countries at: https://www.ntppool.org/zone/@
  sntp_server_1: "192.168.11.1"
  sntp_server_2: "192.168.10.1"
  # Enables faster network connections, with last connected SSID being connected to and no full scan for SSID being undertaken
  wifi_fast_connect: "false"
  # Define logging level: NONE, ERROR, WARN, INFO, DEBUG (Default), VERBOSE, VERY_VERBOSE
  log_level: "INFO"
  # Enable or disable the use of IPv6 networking on the device
  ipv6_enable: "false"

esphome:
  name: "${name}"
  friendly_name: "${friendly_name}"
  comment: "${device_description}"
  name_add_mac_suffix: false
  min_version: 2024.6.0

esp8266:
  board: esp01_1m

# BEGIN SENSORS
sensor:

# BEGIN PLATFORM SENSORS
  - platform: uptime
    name: "Uptime Sensor"
    id: uptime_sensor
    entity_category: diagnostic
    internal: true

  - platform: wifi_signal
    name: "WiFi Signal dB"
    id: wifi_signal_db
    update_interval: ${sensor_update_interval}
    entity_category: "diagnostic"

  - platform: copy
    source_id: wifi_signal_db
    name: "WiFi Signal Percent"
    filters:
      - lambda: return min(max(2 * (x + 100.0), 0.0), 100.0);
    unit_of_measurement: "Signal %"
    entity_category: "diagnostic"
    device_class: ""

# BEGIN BINARY SENSORS
binary_sensor:
  - platform: status
    name: "Status"
    entity_category: diagnostic

# BEGIN TEXT SENSORS
text_sensor:
  - platform: wifi_info
    ip_address:
      name: "IP Address"
      entity_category: diagnostic
    ssid:
      name: "Connected SSID"
      entity_category: diagnostic
    mac_address:
      name: "Mac Address"
      entity_category: diagnostic

# Enable logging
logger:
  baud_rate: 0 #need this to free up UART pins

# SET UART off
uart:
  baud_rate: 115200 # speed to STC15L101EW
  tx_pin: GPIO1
  rx_pin: GPIO3

# BEGIN Switch
switch:
  - platform: uart
    name: "Zone 1 Back Yard"
    icon: "mdi:sprinkler-variant"
    data:
      turn_on: [0xA0, 0x01, 0x01, 0xA2]
      turn_off: [0xA0, 0x01, 0x00, 0xA1]
  - platform: uart
    name: "Zone 2 Back Yard"
    icon: "mdi:sprinkler-variant"
    data:
      turn_on: [0xA0, 0x02, 0x01, 0xA3]
      turn_off: [0xA0, 0x02, 0x00, 0xA2]
  - platform: uart
    name: "Zone 3 Back Yard"
    icon: "mdi:sprinkler-variant"
    data:
      turn_on: [0xA0, 0x03, 0x01, 0xA4]
      turn_off: [0xA0, 0x03, 0x00, 0xA3]
  - platform: uart
    name: "Zone 4 Back Yard"
    icon: "mdi:sprinkler-variant"
    data:
      turn_on: [0xA0, 0x04, 0x01, 0xA5]
      turn_off: [0xA0, 0x04, 0x00, 0xA4]
# switch:
#  - platform: template
#    name: 'BackYard Sprinklers Zone 1'
#    id: relay1
#    icon: "mdi:sprinkler-variant"
#   turn_on_action:
#      - uart.write: [0xA0, 0x01, 0x01, 0xA2]
#    turn_off_action:
#      - uart.write: [0xA0, 0x01, 0x00, 0xA1]
#    optimistic: true

#  - platform: template
#    name: 'BackYard Sprinklers Zone 2'
#    id: relay2
#    icon: "mdi:sprinkler-variant"
#    turn_on_action:
#      - uart.write: [0xA0, 0x02, 0x01, 0xA3]
#    turn_off_action:
#      - uart.write: [0xA0, 0x02, 0x00, 0xA2]
#    optimistic: true#

#  - platform: template
#    name: 'BackYard Sprinklers Zone 3'
#    id: relay3
#    icon: "mdi:sprinkler-variant"
#    turn_on_action:
#      - uart.write: [0xA0, 0x03, 0x01, 0xA4]
#   turn_off_action:
#      - uart.write: [0xA0, 0x03, 0x00, 0xA3]
#    optimistic: true

#  - platform: template
#    name: 'BackYard Sprinklers Zone 4'
##    id: relay4
#   icon: "mdi:sprinkler-variant"
#    turn_on_action:
#      - uart.write: [0xA0, 0x04, 0x01, 0xA5]
#    turn_off_action:
#      - uart.write: [0xA0, 0x04, 0x00, 0xA4]
#    optimistic: true

    
# Enable Home Assistant API
api:

ota:
  - platform: esphome

# Webserver
web_server:  
  port: 80

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  min_auth_mode: WPA2
  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Esp8266-Esp01-FBHS"
    password: "WKA8D0rnflrq"
