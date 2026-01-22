# Integrating MAYARA with Freeboard-SK

## Current Situation

- **MAYARA** runs on port `6502` (or `3001` in docker) and provides:
  - Web interface with radar viewer at `http://localhost:6502/viewer.html?id=radar-1`
  - WebSocket API for radar data
  - REST API at `http://localhost:6502/v1/api/radars`

- **SignalK/Freeboard** runs on port `3000` and provides:
  - Chart plotter interface
  - Navigation data
  - AIS, waypoints, etc.

- **Current limitation**: MAYARA does NOT send radar data to SignalK. They run independently.

## Option 1: Use MAYARA Viewer Directly (Simplest)

Just open the MAYARA viewer in a separate browser tab or window:

```
http://localhost:6502/viewer.html?id=radar-1
```

This works immediately and shows the radar PPI (Plan Position Indicator).

## Option 2: Embed MAYARA Viewer in Freeboard (Recommended)

### Method A: Using Freeboard Widget/Iframe

Freeboard-SK supports custom widgets. You can create a widget that embeds the MAYARA viewer:

1. **Access Freeboard-SK configuration**:
   - Go to `http://localhost:3000` (or your SignalK port)
   - Navigate to Freeboard-SK settings

2. **Add a custom widget/panel**:
   - Create a new panel/widget
   - Use an iframe to embed MAYARA viewer:
   ```html
   <iframe 
     src="http://localhost:6502/viewer.html?id=radar-1" 
     width="100%" 
     height="600px"
     frameborder="0">
   </iframe>
   ```

### Method B: Side-by-Side Layout

Open both in separate browser windows and arrange them side-by-side:
- Left: Freeboard-SK at `http://localhost:3000`
- Right: MAYARA viewer at `http://localhost:6502/viewer.html?id=radar-1`

### Method C: Browser Extension/Bookmark

Create a browser bookmark that opens both:
```javascript
javascript:(function(){
  window.open('http://localhost:3000', 'freeboard');
  window.open('http://localhost:6502/viewer.html?id=radar-1', 'radar');
})();
```

## Option 3: Create SignalK Plugin (Advanced)

To actually send radar data TO SignalK, you would need to:

1. **Create a SignalK plugin** that:
   - Connects to MAYARA's WebSocket API (`ws://localhost:6502/v1/api/spokes/radar-1`)
   - Receives radar spoke data
   - Converts it to SignalK format
   - Publishes to SignalK paths like `sensors.radar.*`

2. **Modify Freeboard-SK** to:
   - Subscribe to radar data from SignalK
   - Render radar overlay on the chart

This requires significant development work.

## Option 4: Use Docker Demo Setup

The demo docker already sets up both services:

```bash
cd demo
./build.sh
```

This will:
- Start MAYARA on port 3001
- Start SignalK on port 3000
- Automatically replay the pcap file

Then access:
- Freeboard: `http://localhost:3000`
- MAYARA viewer: `http://localhost:3001/viewer.html?id=radar-1`

## Recommended Approach

For now, **Option 1 or 2A** (embed iframe) is the most practical:

1. **Use MAYARA's built-in viewer** - it's fully functional and displays radar data
2. **Embed it in Freeboard** if you want everything in one interface
3. **Keep them separate** if you prefer dedicated windows

The radar data is already being processed and displayed correctly by MAYARA. The integration with SignalK would be nice-to-have but isn't necessary for viewing radar data.

## Testing Your Setup

1. **Verify MAYARA is working**:
   ```bash
   curl http://localhost:6502/v1/api/radars
   ```
   Should return JSON with radar info.

2. **Open viewer**:
   ```
   http://localhost:6502/viewer.html?id=radar-1
   ```
   Replace `radar-1` with the actual ID from the API.

3. **Check SignalK**:
   ```
   http://localhost:3000
   ```
   Should show Freeboard interface.

## Future Development

If you want to build a SignalK plugin to send radar data:

1. Study SignalK plugin development: https://github.com/SignalK/signalk-server
2. Create a plugin that connects to MAYARA's WebSocket
3. Convert RadarMessage protobuf to SignalK delta format
4. Publish to SignalK paths like `sensors.radar.sweep` or `sensors.radar.returns`

This would allow Freeboard-SK to natively display radar overlays on charts, but requires custom development.
