# Home Assistant Blueprints

This repository contains Home Assistant blueprints for automations.

## Timer-Based Lighting Automation

**Version:** v2026.01.26  
**Author:** Dima Tokar  
**Minimum Home Assistant Version:** 2025.1.0

### Description

Turn on lights based on a trigger (e.g., motion) and keep them on using a Timer helper. The timer restarts with each new trigger event. Lights turn off when the timer finishes. Optionally restart the timer when any controlled light is turned on (manually or by other automations). Supports optional conditions for turning on/off and works with Lights, Areas, Devices, and Labels.

Read more about it on my blog: [https://www.dima.pm/timer-based-lighting-blueprint-for-home-assistant/](https://www.dima.pm/timer-based-lighting-blueprint-for-home-assistant/)

### Features

- **Flexible Triggers**: Use motion sensors, door sensors, or any other state-based entities to trigger the automation
- **Timer Helper Integration**: Leverages Home Assistant's built-in timer helper to manage light duration
- **Automatic Restart**: Timer automatically restarts when new triggers are detected
- **Timer-Only Triggers**: Optionally designate certain sensors to only restart the timer without turning on lights (useful for mmWave sensors that should maintain but not activate lights)
- **Optional Manual Restart**: Configure the timer to restart when lights are turned on manually or by other automations
- **Conditional Execution**: Support for global conditions, turn-on conditions, and turn-off conditions
- **Multi-Target Support**: Works with individual lights, areas, devices, and labels
- **Debug Mode**: Built-in debug logging for troubleshooting

### Import URL

```
https://github.com/dimatx/home-assistant/blob/main/blueprints/timer_driven_lighting_blueprint.yaml
```

### Configuration Options

#### Required Configuration

- **Timer Entity**: The timer helper that tracks the duration for the lights to stay on
- **Lights to Control**: The lights, areas, devices, or labels to manage
- **Trigger Entities**: Sensors or entities (e.g., motion sensors, door sensors) that trigger the timer
- **Trigger States**: Comma-separated list of states that trigger the timer (e.g., 'on, open'). Defaults to 'on'

#### Conditions

- **Global Conditions**: Conditions that must be met for ANY action (turn on or off) to run
- **Turn ON Conditions**: Additional conditions that must be met specifically to turn the lights ON
- **Turn OFF Conditions**: Additional conditions that must be met specifically to turn the lights OFF

#### Other Settings

- **Timer Duration (Optional)**: If specified, this duration overrides the default duration configured in the timer helper
- **Restart Timer on Light On**: If enabled, the timer will restart whenever any of the monitored lights is turned on (manually or by other automations)
- **Lights to Monitor**: Specify which light entities should restart the timer when turned on (only used if "Restart Timer on Light On" is enabled)
- **Timer-Only Trigger Entities**: Trigger entities listed here will restart the timer but will NOT turn on lights if they're off. Useful for mmWave sensors that should only maintain lights, not activate them. These entities must also be listed in 'Trigger Entities'
- **Debug Mode**: If enabled, debug information will be logged to the system log

### Usage

1. Create a Timer helper in Home Assistant (Settings > Devices & Services > Helpers > Create Helper > Timer)
2. Import this blueprint using the URL above
3. Create a new automation based on the blueprint
4. Configure the required settings:
   - Select your timer helper
   - Choose the lights you want to control
   - Select your trigger entities (e.g., motion sensors)
   - Set the trigger states (default is 'on')
5. Optionally configure conditions and other settings
6. Save and enable the automation

### How It Works

1. When a trigger entity changes to one of the specified trigger states, the timer starts/restarts
2. If the timer was idle, the lights turn on immediately
3. The timer continues to restart with each new trigger event
4. When the timer expires (finishes), the lights turn off
5. If "Restart Timer on Light On" is enabled, manually turning on a monitored light will also restart the timer

### Example Use Cases

- **Motion-Activated Lighting**: Turn on bathroom lights when motion is detected and keep them on for 5 minutes, restarting the timer with each motion event
- **Occupancy-Based Lighting**: Use occupancy sensors to keep lights on while a room is occupied
- **Hybrid Sensor Setup**: Use a PIR sensor to turn on lights and an mmWave sensor to keep them on—add the mmWave to Timer-Only Trigger Entities so it only maintains the timer without activating lights on its own
- **Multi-Sensor Control**: Combine multiple sensors (motion + door) to control lights in an entryway
- **Conditional Automation**: Only turn on lights during nighttime hours using time-based conditions
