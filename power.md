---
layout: default
title: Power Configuration
sidebar_label: Power
description: Settings for advanced users who want to adjust the power configuration on their Meshtastic device.
---

import Admonition from "@theme/Admonition";
import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

:::info Use Device Config first
Power settings are advanced configuration, most users should choose a role under [Device Config](/docs/configuration/radio/device) to manage power for their device and shouldn't ever need to adjust these settings.
:::

The power config options are: Power Saving, Shutdown after losing power, ADC Multiplier Override, Wait Bluetooth Interval, Light Sleep Interval, Minimum Wake Interval, and Device Battery INA2xx Address. Power config uses an admin message sending a `Config.Power` protobuf.

## Power Config Values

### Power Saving

:::warning
If enabled, modifications to settings can be made by waking the device through pressing the user button, resetting, or through the [admin channel](/docs/configuration/remote-admin/) for remote administration.
:::

When activated, this feature disables Bluetooth, Serial, WiFi, and the device's screen to conserve power. This is particularly beneficial for devices relying on low-current power sources, like solar panels. For details on which device roles have this feature enabled by default, please check the [Device Config](/docs/configuration/radio/device) section.

### Shutdown after losing power

Automatically shut down a device after a defined time period if power is lost.

Before setting this, ensure your board correctly detects when it is connected to external power (the "🔌" icon in the app). Otherwise, your device could shut down even when power is present.
Many devices detect power status correctly, however for some devices, you will need to modify the ADC Multiplier (below).

### ADC Multiplier Override

Ratio of voltage divider for battery pin e.g. 3.20 (R1=100k, R2=220k)

Overrides the ADC_MULTIPLIER defined in the firmware device variant file for battery voltage calculation.

Should be set to a floating point value between 2 and 6

#### Calibration Process ([Attribution](https://wiki.uniteng.com/en/meshtastic/nano-g1-explorer#calibration-process))

1. Install the rechargeable battery.
2. Charge the battery until full. Indication of this state may vary depending on device. At this point, the battery voltage should be 4.2V +-1%.
3. Input the "Battery Charge Percent" displayed on the screen or in your connected app into the calculator below.
4. If "Battery Charge Percent" (e.g., B 3.82V 60%) is not displayed on the screen, it means that the default value of "Operative Adc Multiplier" is too high. Lower the "Operative Adc Multiplier" to a smaller number (it is recommended to decrease by 0.1) until the screen displays "Battery Charge Percent". Enter the current "Operative Adc Multiplier" in use into the "Operative Adc Multiplier" field in the calculator. Also, input the "Battery Charge Percent" displayed on the screen into the calculator.
5. Click the "Calculate" button to compute the "Calculated New Operative Adc Multiplier", and set it as the new "Operative Adc Multiplier" for the device.

<div class="no-print">
  <details>
    <summary>ADC Calculator</summary>
    <div>
      <Admonition type="info">
        <p>This calibration method only maps 4.2V to Battery Charge Percent 100%, and does not address the potential non-linearities of the ADC.</p>
      </Admonition>
      <table>
        <tr>
          <td>Device:</td>
          <td>
            <select id="deviceSelect" onchange="updateAdcMultiplier()">
              <option value="chatter2" data-multiplier="5.0">chatter2</option>
              <option value="diy" data-multiplier="1.85">diy</option>
              <option value="esp32-s3-pico" data-multiplier="3.1">esp32-s3-pico</option>
              <option value="heltec_v1" data-multiplier="3.2">heltec_v1</option>
              <option value="heltec_v2" data-multiplier="3.2">heltec_v2</option>
              <option value="heltec_v3" data-multiplier="4.9">heltec_v3</option>
              <option value="heltec_wsl_v3" data-multiplier="4.9">heltec_wsl_v3</option>
              <option value="heltec_wireless_paper" data-multiplier="2.0">heltec_wireless_paper</option>
              <option value="heltec_wireless_tracker" data-multiplier="4.9">heltec_wireless_tracker</option>
              <option value="lora_isp4520" data-multiplier="1.436">lora_isp4520</option>
              <option value="m5stack_coreink" data-multiplier="5.0">m5stack_coreink</option>
              <option value="nano-g1-explorer" data-multiplier="2.0">nano-g1-explorer</option>
              <option value="nano-g2-ultra" data-multiplier="2.0">nano-g2-ultra</option>
              <option value="picomputer-s3" data-multiplier="3.1">picomputer-s3</option>
              <option value="rak4631" data-multiplier="1.73">rak4631</option>
              <option value="rpipico" data-multiplier="3.1">rpipico</option>
              <option value="rpipicow" data-multiplier="3.1">rpipicow</option>
              <option value="station-g1" data-multiplier="6.45">station-g1</option>
              <option value="station-g2" data-multiplier="4.0">station-g2</option>
              <option value="tlora_v2_1_16" data-multiplier="2.0">tlora_v2_1_16</option>
              <option value="tlora_v2_1_18" data-multiplier="2.11">tlora_v2_1_18</option>
              <option value="tlora_t3s3_v1" data-multiplier="2.11">tlora_t3s3_v1</option>
              <option value="t-deck" data-multiplier="2.11">t-deck</option>
              <option value="t-echo" data-multiplier="2.0">t-echo</option>
            </select>
          </td>
        </tr>
        <tr>
          <td>Battery Voltage (V):</td>
          <td><input type="text" id="batteryVoltage" value="3.82" /></td>
        </tr>
        <tr>
          <td>Battery Charge Percent:</td>
          <td><input type="text" id="batteryChargePercent" value="65" /></td>
        </tr>
        <tr>
          <td>Current Adc Multiplier:</td>
          <td><input type="text" id="operativeAdcMultiplier" value="5.0" /></td>
        </tr>
        <tr>
          <td>Calculated New Operative Adc Multiplier:</td>
          <td><input type="text" id="newOperativeAdcMultiplier" value="5.0" disabled="disabled" /></td>
        </tr>
        <tr>
          <td></td>
          <td>
            <button class="button button--outline button--lg cta--button" onclick="calculateNewMultiplier()">Calculate</button>
          </td>
        </tr>
      </table>
      <script>
        function updateAdcMultiplier() {
          var select = document.getElementById('deviceSelect');
          var multiplier = select.options[select.selectedIndex].getAttribute('data-multiplier');
          document.getElementById('operativeAdcMultiplier').value = multiplier;
        }

        function calculateNewMultiplier() {
          var batteryVoltage = parseFloat(document.getElementById('batteryVoltage').value);
          var batteryChargePercent = parseFloat(document.getElementById('batteryChargePercent').value);
          var currentAdcMultiplier = parseFloat(document.getElementById('operativeAdcMultiplier').value);

          if (isNaN(batteryVoltage) || batteryVoltage <= 0 || isNaN(batteryChargePercent) || batteryChargePercent < 0 || batteryChargePercent > 100 || isNaN(currentAdcMultiplier) || currentAdcMultiplier < 2 || currentAdcMultiplier > 6) {
            alert("Please enter valid numbers within the specified ranges.");
            return;
          }

          var targetVoltage = 4.19;
          var newAdcMultiplier = currentAdcMultiplier * (targetVoltage / batteryVoltage);

          document.getElementById('newOperativeAdcMultiplier').value = newAdcMultiplier.toFixed(3);
        }
      </script>
    </div>
  </details>
</div>

### Wait Bluetooth Interval

How long to wait before turning off BLE in no Bluetooth states

`0` for default of 1 minute

### Light Sleep Interval

:::info ESP32 Only
This setting only applies to ESP32-based boards, it will have no effect on nRF52/RP2040 boards.
:::

In light sleep the CPU is suspended, LoRa radio is on, BLE is off and GPS is on

`0` for default of five minutes

### Minimum Wake Interval

While in light sleep when we receive packets on the LoRa radio we will wake and handle them and stay awake in no Bluetooth mode for this interval

`0` for default of 10 seconds

### Device Battery INA2xx Address

If an INA-2XX device is auto-detected on one of the I2C buses at the specified address, it will be used as the authoritative source for reading device battery level voltage. Setting is ignored for devices with PMUs (e.g. T-beams)

:::tip[Convert hexadecimal to decimal]
I2C addresses are normally represented in hexadecimal and will require conversion to decimal in order to set via Meshtastic clients. For example the I2C address of 0x40 converted to decimal is 64.
:::

## Power Config Client Availability

<Tabs
groupId="settings"
defaultValue="cli"
values={[
{label: 'Android', value: 'android'},
{label: 'Apple', value: 'apple'},
{label: 'CLI', value: 'cli'},
{label: 'Web', value: 'web'},
]}>
<TabItem value="android">

#### Android

:::info

Power Config options are available for Android.

1. Open the Meshtastic App
2. Navigate to: **Vertical Ellipsis (3 dots top right) > Radio Configuration > Power**

:::

</TabItem>
<TabItem value="apple">

#### Apple

:::info

Select Power config options are available on iOS, iPadOS and macOS at Settings > Device Configuration > Power.

:::

</TabItem>
<TabItem value="cli">

#### CLI

:::info

All Power config options are available in the python CLI.

:::

|               Setting                |         Acceptable Values          |               Default               |
| :----------------------------------: | :--------------------------------: | :---------------------------------: |
|        power.is_power_saving         |          `true`, `false`           |               `false`               |
| power.on_battery_shutdown_after_secs |        `integer` (seconds)         |        Default of `0` is off        |
|    power.adc_multiplier_override     |    `2-4` (floating point value)    | Default of `0` uses firmware values |
|      power.wait_bluetooth_secs       |        `integer` (seconds)         |     Default of `0` is 1 minute      |
|            power.ls_secs             |        `integer` (seconds)         |     Default of `0` is 5 minutes     |
|         power.min_wake_secs          |        `integer` (seconds)         |    Default of `0` is 10 seconds     |
|   power.device_battery_ina_address   | `integer` (I2C address as decimal) |  Default of `0` is no address set   |

:::tip

Because the device will reboot after each command is sent via CLI, it is recommended when setting multiple values in a config section that commands be chained together as one.

```shell title="Example:"
meshtastic --set power.is_power_saving true --set power.on_battery_shutdown_after_secs 120
