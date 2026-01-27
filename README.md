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

At a high level, this blueprint manages lights based on activity detection and a timer:

1. **Activity detected** → Timer starts/restarts, lights turn on (if conditions are met)
2. **Continued activity** → Timer keeps restarting, lights stay on
3. **No activity** → Timer expires, lights turn off (if conditions are met)

### Example Use Cases

- **Motion-Activated Lighting**: Turn on bathroom lights when motion is detected and keep them on for 5 minutes, restarting the timer with each motion event
- **Occupancy-Based Lighting**: Use occupancy sensors to keep lights on while a room is occupied
- **Hybrid Sensor Setup**: Use a PIR sensor to turn on lights and an mmWave sensor to keep them on—add the mmWave to Timer-Only Trigger Entities so it only maintains the timer without activating lights on its own
- **Multi-Sensor Control**: Combine multiple sensors (motion + door) to control lights in an entryway
- **Conditional Automation**: Only turn on lights during nighttime hours using time-based conditions

---

## Detailed Logic Reference

This section provides a detailed explanation of the blueprint's internal logic for advanced users and troubleshooting.

### Triggers

The automation responds to four types of events:

| Trigger ID | Event | Description |
|------------|-------|-------------|
| `timer_active` | Timer goes from idle → active | Timer just started |
| `timer_idle` | Timer goes to idle | Timer finished or was cancelled |
| `entity_triggered` | Any trigger entity changes state | Motion detected, door opened, etc. |
| `light_turned_on` | Monitored light turns on | External light activation (manual, other automations) |

### Action Branches

The automation uses a `choose` block with 5 branches, evaluated in order:

#### Branch 1: Activity Detected → Start/Restart Timer

**When:** `entity_triggered` fires AND state matches `trigger_states` AND (timer is already active OR `turn_on_conditions` are met)

**Actions:**
1. If timer is idle AND trigger is NOT in `timer_only_trigger_entities` → Turn on lights
2. Start/restart the timer

**Key behavior:** Timer-only triggers (like mmWave sensors) will restart the timer but skip the light turn-on step. This allows presence sensors to maintain lights without activating them.

#### Branch 1b: Light Turned On → Restart Timer

**When:** `light_turned_on` fires AND `restart_on_light_on` is enabled AND the light was turned on externally (not by this automation)

**Actions:** Restart the timer

**Key behavior:** Uses context comparison to prevent self-triggering loops.

#### Branch 2: Timer Active → Turn Lights ON (External Start Only)

**When:** `timer_active` fires AND timer was started externally (not by this automation) AND `turn_on_conditions` are met

**Actions:** Turn on lights

**Key behavior:** This branch only fires when the timer is started by something other than this automation (e.g., manually, via script, or another automation). When Branch 1 starts the timer, it handles lights directly, so Branch 2 is skipped using context comparison (`trigger.to_state.context.id != this.context.parent_id`).

#### Branch 3: Timer Idle → Turn Lights OFF

**When:** `timer_idle` fires AND `turn_off_conditions` are met

**Actions:** Turn off lights

**Key behavior:** This is the primary turn-off path when the timer expires naturally.

#### Branch 4: Activity Detected & Timer Already Idle → Turn Lights OFF

**When:** `entity_triggered` fires AND timer is already idle AND `turn_off_conditions` are met

**Actions:** Turn off lights

**Key behavior:** This handles edge cases where the timer expired but lights stayed on due to a blocking `turn_off_condition` (e.g., mmWave still detecting presence). When that condition clears (e.g., mmWave goes off), this branch catches the state change and turns off the lights.

### Condition Types Explained

| Condition Type | Scope | Example Use Case |
|----------------|-------|------------------|
| **Global Conditions** | All actions (on and off) | "Light automations enabled" toggle, home occupied |
| **Turn ON Conditions** | Only light turn-on | Night time only, sleep mode off |
| **Turn OFF Conditions** | Only light turn-off | No presence detected, not watching TV |

### Timer-Only Triggers: When to Use

Use `timer_only_trigger_entities` when you have sensors that should **maintain** lights but not **activate** them:

| Sensor Type | Behavior | Recommended Setting |
|-------------|----------|---------------------|
| PIR motion sensor | Quick response, clears when no motion | Normal trigger (turns on lights) |
| mmWave presence sensor | Detects stationary presence, may get "stuck" | Timer-only (maintains lights) |
| Door sensor | Instant trigger on open | Normal trigger (turns on lights) |

**Typical hybrid setup:**
- PIR in `trigger_entities` → turns on lights when you enter
- mmWave in both `trigger_entities` AND `timer_only_trigger_entities` → keeps timer running while you're present, but won't turn on lights by itself

### Context-Based Self-Detection

The blueprint uses Home Assistant's context system to detect when events are caused by itself vs external sources:

- **Branch 1b** checks `trigger.to_state.context.id != this.context.id` to ignore lights it turned on itself
- **Branch 2** checks `trigger.to_state.context.id != this.context.parent_id` to skip when it started the timer

This prevents loops and ensures each branch only handles its intended scenarios.
