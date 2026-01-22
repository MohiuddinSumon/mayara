# How MAYARA Connects to SignalK

## The Connection Process

When MAYARA starts, it automatically tries to find SignalK using **mDNS (multicast DNS)**:

1. **Service Discovery**: MAYARA searches for SignalK service named `_signalk-tcp._tcp.local.` on your local network
2. **Automatic Discovery**: It uses mDNS to find any SignalK server running on the same network
3. **Connection**: Once found, it connects via TCP and subscribes to navigation data:
   - `navigation.headingTrue` (heading)
   - `navigation.position` (latitude/longitude)
   - `navigation.speedOverGround` (speed)
   - `navigation.courseOverGroundTrue` (course)

## What Happens When SignalK is NOT Running?

**Good news**: MAYARA works perfectly fine without SignalK!

When SignalK is not available:

1. **MAYARA keeps trying**: It continuously searches for SignalK in the background (every few seconds)
2. **No errors**: It doesn't crash or fail - it just logs debug messages
3. **Radar still works**: The radar processes and displays data normally
4. **Navigation data is optional**: Functions return `None` when data isn't available

## What You're Seeing

Since you're only running MAYARA and tcpreplay:

- ✅ **Radar detection**: Works (you see the radar detected)
- ✅ **Radar data processing**: Works (data is being processed)
- ✅ **Web viewer**: Works (you can view the radar)
- ⚠️ **Navigation data**: Not available (no SignalK running)
  - Heading: `None` (radar displays relative to boat, not true north)
  - Position: `None` (no GPS coordinates)
  - Speed/Course: `None`

## The Code Flow

```rust
// MAYARA tries to find SignalK
find_mdns_service() 
  → Searches for "_signalk-tcp._tcp.local."
  → If found: connects and subscribes
  → If not found: keeps retrying (logs debug messages)

// Navigation data functions return Option<f64>
get_heading_true() → Option<f64>  // None if SignalK not available
get_position() → Option<GeoPosition>  // None if SignalK not available

// Radar still works - these are optional
to_protobuf_spoke() 
  → Uses heading if available (None otherwise)
  → Uses position if available (None otherwise)
```

## Why This Design?

Navigation data (heading, position) is **optional** because:

1. **Relative mode**: Radar can display relative to the boat's heading (0° = ahead)
2. **No GPS needed**: Radar works without knowing your exact position
3. **True north mode**: If heading is available, radar can show true bearings
4. **Chart overlay**: Position is only needed if overlaying radar on charts

## Checking if SignalK is Connected

Look at your MAYARA server logs:

**If SignalK is found:**
```
Listening to Signal K data from 192.168.1.100:3000
```

**If SignalK is NOT found:**
```
SignalK find_service (re)start
find_service restart on result: ...
```
(These are debug messages - MAYARA keeps trying but doesn't fail)

## Summary

- **MAYARA automatically searches** for SignalK via mDNS
- **It works fine without SignalK** - navigation data is optional
- **Your radar is working correctly** even without SignalK
- **If you want navigation data**, start SignalK server and MAYARA will automatically connect

The radar display you're seeing is working in "relative mode" - showing data relative to the boat's heading (0° = straight ahead) rather than true north.
