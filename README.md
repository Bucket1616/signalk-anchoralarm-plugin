# signalk-anchoralarm-plugin

[![Greenkeeper badge](https://badges.greenkeeper.io/sbender9/signalk-anchoralarm-plugin.svg)](https://greenkeeper.io/)

SignalK Node Server Anchor Alarm Plugin

Then use WilhelmSK to set the alarm (https://itunes.apple.com/us/app/wilhelmsk/id1150499484?mt=8)

If not using WilhelmSK, you can setup the alarm using the WebApp or the REST API.

## Web App

Point your Web Browser to http://[signalk-server-ip-address]:[port-number]/signalk-anchoralarm-plugin/

If you wish to have the satellite or openseamaps view enabled by default add the following

| OpenStreetMap | Satellite | OpenSeaMap | Url String |
| ------------- | --------- | ---------- | -----------|
| X | - | - | / |
| X | - | X | /?openseamap |
| - | X | - | /?satellite |
| - | X | X | /?satellite&openseamap |

Note that you must be logged in to SignalK UI for this to work.

When a depth transducer is configured the plugin will default to an anchor alarm of Dx5. If no depth transducer can be found the web app will prompt for the anchor alarm radius when the anchor is droped.

## REST API

### When you drop the anchor in the water, Call dropAnchor:


```
curl -X POST -H "Content-Type: application/json" -d '{}' http://localhost:3000/plugins/anchoralarm/dropAnchor
```

### After you have let the anchor rode out, call setRadius. This will calculate and set the alarm radius based on the vessels current position.

```
curl -X POST -H "Content-Type: application/json" -d '{}' http://localhost:3000/plugins/anchoralarm/setRadius
```

### Alternately, after you have let the anchor rode out, call setRodeLength. This will calculate and set the alarm radius based on rode length, depth and bow height.

```
curl -X POST -H "Content-Type: application/json" -d '{"length": 30}' http://localhost:3000/plugins/anchoralarm/setRodeLength
```


### You can adjust the radius (in meters) via:

```
curl -X POST -H "Content-Type: application/json" -d '{"radius": 30}' http://localhost:3000/plugins/anchoralarm/setRadius
```

### When you raise the anchor, call raiseAnchor.

```
curl -X POST -H "Content-Type: application/json" -d '{}' http://localhost:3000/plugins/anchoralarm/raiseAnchor
```

### If you need to set the anchor position after you have already let the rode out, it can esitmate the andchor position based on heading, depth and rode length. If "anchorDepth" is left out, then the current depthFromSurface will be used if available.

```
curl -X POST -H "Content-Type: application/json" -d '{"anchorDepth": 3, "rodeLength":30}' http://localhost:3000/plugins/anchoralarm/setManualAnchor
```

## Action Handlers

The plugin registers action handlers that allow other parts of the Signal K server (such as apps or other plugins) to control the anchor alarm through PUT requests to Signal K paths. These handlers provide a standardized Signal K interface for anchor operations.

### Available Action Handlers

#### navigation.anchor.position
- **Path**: `vessels.self.navigation.anchor.position`
- **Purpose**: Set or clear the anchor position
- **Value**: Position object with `latitude`, `longitude`, and optionally `altitude` properties, or `null` to raise the anchor
- **Behavior**: 
  - When a position is provided, sets the anchor position and starts monitoring if a radius is configured
  - When `null` is provided, raises the anchor and stops monitoring
- **Example PUT request**:
```json
{
  "context": "vessels.self",
  "requestId": "12345",
  "put": {
    "path": "navigation.anchor.position",
    "value": {
      "latitude": 37.7749,
      "longitude": -122.4194,
      "altitude": -5.2
    }
  }
}
```
- **Example curl command**:
```bash
curl -X PUT -H "Content-Type: application/json" \
  -d '{"value":{"latitude":37.7749,"longitude":-122.4194,"altitude":-5.2}}' \
  http://localhost:3000/signalk/v1/api/vessels/self/navigation/anchor/position
```
- **To raise the anchor (set position to null)**:
```bash
curl -X PUT -H "Content-Type: application/json" \
  -d '{"value":null}' \
  http://localhost:3000/signalk/v1/api/vessels/self/navigation/anchor/position
```

#### navigation.anchor.maxRadius
- **Path**: `vessels.self.navigation.anchor.maxRadius`
- **Purpose**: Set the maximum anchor alarm radius in meters
- **Value**: Number representing radius in meters
- **Behavior**: Sets the alarm radius and starts monitoring if an anchor position is already set
- **Example PUT request**:
```json
{
  "context": "vessels.self",
  "requestId": "12346",
  "put": {
    "path": "navigation.anchor.maxRadius",
    "value": 50
  }
}
```
- **Example curl command**:
```bash
curl -X PUT -H "Content-Type: application/json" \
  -d '{"value":50}' \
  http://localhost:3000/signalk/v1/api/vessels/self/navigation/anchor/maxRadius
```

#### navigation.anchor.rodeLength
- **Path**: `vessels.self.navigation.anchor.rodeLength`
- **Purpose**: Set the anchor rode length and automatically calculate the appropriate alarm radius
- **Value**: Number representing rode length in meters
- **Behavior**: 
  - Sets the rode length value in the Signal K data
  - Automatically calculates and sets the anchor position based on current vessel position, heading, depth, and configuration
  - Starts anchor monitoring with the calculated parameters
- **Example PUT request**:
```json
{
  "context": "vessels.self",
  "requestId": "12347",
  "put": {
    "path": "navigation.anchor.rodeLength",
    "value": 30
  }
}
```
- **Example curl command**:
```bash
curl -X PUT -H "Content-Type: application/json" \
  -d '{"value":30}' \
  http://localhost:3000/signalk/v1/api/vessels/self/navigation/anchor/rodeLength
```

### Using Action Handlers

These action handlers can be triggered by:
- Signal K apps (like WilhelmSK)
- Other plugins
- Direct HTTP PUT requests to the Signal K server's REST API
- WebSocket PUT messages

The handlers provide a more standardized interface compared to the plugin-specific REST endpoints, following Signal K conventions for data paths and action handling.


