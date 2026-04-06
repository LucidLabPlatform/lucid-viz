# LUCID Foraging Arena — GAMA Digital Twin

Real-time projected overlay for the LUCID foraging experiment. Subscribes to LUCID MQTT telemetry and renders a 2D arena view showing robot position, puck locations, ArUco corners, movement trail, and experiment status.

## Prerequisites

- [GAMA Platform](https://gama-platform.org/) 1.9+
- LUCID Central Command stack running (EMQX broker accessible)
- TouchDesigner (for projection mapping)

## Setup

1. Open GAMA Platform
2. Import this project: **File > Import > Existing GAMA project** and select this folder
3. Open `models/foraging_arena.gaml`
4. Configure parameters in the experiment panel:
   - **MQTT Broker**: IP/hostname of the EMQX broker (default: `localhost`)
   - **MQTT Port**: Broker port (default: `1883`)
   - **MQTT User / Password**: LUCID MQTT credentials
   - **Robot Agent ID**: LUCID agent ID for the foraging robot (default: `ra-lab-c5`)
   - **OptiTrack Agent ID**: LUCID agent ID for OptiTrack bridge (default: `optitrack`)
   - **Rigid Body Name**: Name of the ROSbot rigid body in Motive (default: `rosbot`)
   - **Arena Width / Height**: Physical arena dimensions in metres
5. Run the `ForagingArena` experiment

## TouchDesigner Integration

1. Run GAMA in fullscreen mode (click the fullscreen button in the display toolbar)
2. In TouchDesigner, add a **Window In TOP** targeting the GAMA window
3. Optionally add a **Crop TOP** to trim any GAMA UI chrome
4. Feed into your existing projection mapping chain

## MQTT Topics Subscribed

| Data | Topic Pattern |
|------|--------------|
| Puck positions | `lucid/agents/{robot_agent_id}/components/ros_bridge/telemetry/puck_registry` |
| Corner markers | `lucid/agents/{robot_agent_id}/components/ros_bridge/telemetry/aruco_registry` |
| Foraging state | `lucid/agents/{robot_agent_id}/components/ros_bridge/telemetry/foraging_status` |
| Robot pose | `lucid/agents/{optitrack_agent_id}/components/ros_bridge/telemetry/vrpn_client_node_{rigid_body_name}_pose` |

## Testing Without Hardware

Publish test messages to EMQX to verify the display:

```bash
# Puck registry
mosquitto_pub -h localhost -t "lucid/agents/ra-lab-c5/components/ros_bridge/telemetry/puck_registry" \
  -m '{"value":{"pucks":[{"id":1,"color":1,"status":0,"x":1.0,"y":1.5},{"id":2,"color":2,"status":0,"x":2.0,"y":2.0}]}}'

# Robot pose
mosquitto_pub -h localhost -t "lucid/agents/optitrack/components/ros_bridge/telemetry/vrpn_client_node_rosbot_pose" \
  -m '{"value":{"pose":{"position":{"x":1.5,"y":1.5,"z":0.0},"orientation":{"x":0.0,"y":0.0,"z":0.0,"w":1.0}}}}'

# Foraging status
mosquitto_pub -h localhost -t "lucid/agents/ra-lab-c5/components/ros_bridge/telemetry/foraging_status" \
  -m '{"value":{"data":"{\"state\":\"running\",\"mode\":0,\"elapsed_s\":45.2}"}}'
```
