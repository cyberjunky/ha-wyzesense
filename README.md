# Home Assistant - WYZE Sense Component

> Special thanks to [HcLX](https://hclxing.wordpress.com) and his work on [WyzeSensePy](https://github.com/HclX/WyzeSensePy) which is the core of this component. His reverse engineering talents and subsequent development of WyzeSensePy made it quite easy to connect with WYZE sense devices.

*Note: You must be running **Home Assistant 0.92 or later**. If you run an older version you may follow the instructions below but replace binary_sensor.py with [this file](https://gist.github.com/kevinvincent/375a063723ecd8b0b06943e3d28ebc93). This is not guaranteed to receive updates.*

## Installation (HACS)
0. Have [HACS](https://custom-components.github.io/hacs/installation/manual/) installed, this will allow you to easily update
1. Add `https://github.com/kevinvincent/ha-wyzesense` as a [custom repository](https://custom-components.github.io/hacs/usage/settings/#add-custom-repositories) as Type: Integration
2. Click install under "Wyze Sense Component"
3. Plug in the WYZE Sense hub (the usb device) into an open port on your device.

## Installation (Manual)
1. Download this repository as a ZIP (green button, top right) and unzip the archive
2. Copy `/custom_components/wyzesense` to your `<config_dir>/custom_components/` directory
   * You will need to create the `custom_components` folder if it does not exist
   * On Hassio the final location will be `/config/custom_components/wyzesense`
   * On Hassbian the final location will be `/home/homeassistant/.homeassistant/custom_components/wyzesense`
3. Plug in the WYZE Sense hub (the usb device) into an open port on your device.

## Configuration
Add the following to your configuration file

```yaml
binary_sensor:
  - platform: wyzesense
    device: "/dev/hidraw0"
```
Most likely your device will be mounted to `/dev/hidraw0`. If you know it is mounted somewhere else then add the appropriate device.

```yaml
binary_sensor:
  - platform: wyzesense
    device: "/dev/hidraw0"
    initial_state:
      77793176: "on"
      77793193: "off"
```
By default, the component will restore the last state of the entity prior to a restart. If sensors change state during a restart, these may not be reflected. In order to combat this you can specify an initial_state for each sensor by mac address that will be set upon a restart.

## Usage
* Restart HA and the sensors you have already bound to the hub (using the wyze app for example) will show up in your entities as `off` with `assumed_state: true` and no `device_class`. These will update and other attributes will be added once the component hears from the sensor for the first time.

* Entities will show up as `binary_sensor.<MAC>` for example (`binary_sensor.777A4656`).
  * As like any other entity you can change the entity id and friendly name from the states page, which will stick even after restarts.

* Call the services below to add and remove sensors from your WYZE sense hub.

* Notes on Individual Sensors
  * Motion
    * State `on`: Motion Detected
    * State `off`: No Motion Detected
    * Wyze motion sensors will keep reporting the `on` state for 40 seconds after the last motion is detected. This is non configurable, but in practice it isn't a big deal and usually makes automations simpler.
  * Door
    * State `on`: Sensor open
    * State `off`: Sensor closed
    * Wyze door sensors will report `off` when the magnetized portion is within ~1 inch of the door sensor body.

## Services
### `wyzesense.scan`
* Call this service and then within 30 seconds, press the button on the side of a sensor with a pin until the red led flashes three times. The sensor will now be bound and show up in your entities.

### `wyzesense.remove`
* Removes a sensor. Make sure you call this service with the correct MAC address of the sensor (which is the string of numbers and possibly letters that looks like `777A4656`). You can find this in the entity's attributes in the developer section.

## Troubleshooting
* Permission denied /dev/hidraw0
  * Additional Information
    * If you see this error on a Hassio installation please follow Reporting an Issue below. It is most likely an issue with your specific setup.
    * This is known to occur on Hassbian. This occurs when the group homeassistant is denied from accessing hidraw devices.
  * Solution
    * Create / Modify the file `/etc/udev/rules.d/99-com.rules` on your machine and insert `KERNEL=="hidraw*", SUBSYSTEM=="hidraw", MODE="0664", GROUP="homeassistant"`
    * Ensure the user running Home Assistant belongs to the homeassistant group

## Reporting an Issue
1. Setup your logger to print debug messages for this component using:
```yaml
logger:
  default: info
  logs:
    custom_components.wyzesense: debug
```
2. Restart HA
3. Verify you're still having the issue
4. File an issue in this Github Repository containing your HA log (Developer section > Info > Load Full Home Assistant Log)
   * The log file can also be found at `/<config_dir>/home-assistant.log`
