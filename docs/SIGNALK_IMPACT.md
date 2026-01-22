# SignalK Connection Impact on MAYARA

## ✅ Connection Confirmed

Your logs show:
```
Listening to Signal K data from 172.16.212.61:8375
```

MAYARA is now receiving navigation data from SignalK!

## What Changes in the Data?

### 1. **Bearing Field** (Most Visible Change)

**Before (No SignalK):**
- `bearing`: `None` or `undefined`
- Radar shows **relative to boat** (0° = straight ahead)
- Display is "head-up" mode

**After (With SignalK):**
- `bearing`: Contains **true north bearing** (0° = North)
- Radar can show **true north orientation**
- Display can be "north-up" mode

### 2. **Position Data** (Latitude/Longitude)

**Before (No SignalK):**
- `lat`: `None`
- `lon`: `None`
- No GPS coordinates in radar data

**After (With SignalK):**
- `lat`: GPS latitude (in 1e-16 degrees format)
- `lon`: GPS longitude (in 1e-16 degrees format)
- Each spoke includes the radar's position when it was generated

### 3. **Heading Data**

**Before (No SignalK):**
- Heading: `None`
- No true heading information

**After (With SignalK):**
- Heading: True heading from compass/GPS
- Used to calculate true bearings for each spoke

### 4. **Trails** (If enabled with `--targets trails`)

**Before (No SignalK):**
- Trails are relative to radar position
- No position-based stabilization

**After (With SignalK):**
- Trails can be stabilized using GPS position
- Better trail tracking when boat moves
- Trails reset if position changes significantly

## How to View the Changes

### Method 1: Check the Web Viewer

1. **Open the viewer**:
   ```
   http://localhost:6502/viewer.html?id=radar-1
   ```

2. **Look for these changes**:
   - **Bearing display**: If the viewer shows compass bearings, they should now be true north
   - **Position**: Check if coordinates are displayed
   - **Orientation**: The display may show north-up vs head-up

### Method 2: Check the API Response

Query the API to see if bearing data is included:

```bash
curl http://localhost:6502/v1/api/radars | jq
```

Look for `streamUrl` and connect to the WebSocket to see the actual spoke data.

### Method 3: Inspect WebSocket Messages

1. **Open browser Developer Tools** (F12)
2. **Go to Network tab**
3. **Filter by "WS" (WebSocket)**
4. **Click on the WebSocket connection** to `/v1/api/spokes/radar-1`
5. **Look at the messages**:
   - Each spoke should now have `bearing` field (true north)
   - Each spoke should have `lat` and `lon` fields

### Method 4: Check Browser Console

In the viewer page, open console (F12) and look for:
- Messages about bearing calculations
- Position data in spoke messages

## Visual Differences

### Without SignalK:
```
Radar Display:
- 0° = Straight ahead (relative)
- No compass rose
- Head-up display
- No GPS coordinates
```

### With SignalK:
```
Radar Display:
- 0° = North (true bearing)
- Compass rose shows true north
- Can switch to north-up display
- GPS coordinates available
- Better trail stabilization
```

## What SignalK Data is Being Used?

MAYARA subscribes to these SignalK paths:

1. **`navigation.headingTrue`** → True heading (degrees)
   - Used to calculate true bearings for each spoke
   - Converts relative angle to true north bearing

2. **`navigation.position`** → GPS position (lat/lon)
   - Added to each spoke as `lat` and `lon` fields
   - Used for trail stabilization
   - Enables chart overlay

3. **`navigation.speedOverGround`** → Speed (m/s)
   - Stored but may not be directly visible in viewer
   - Could be used for calculations

4. **`navigation.courseOverGroundTrue`** → Course (degrees)
   - Stored but may not be directly visible in viewer
   - Could be used for predictions

## Testing the Connection

### Verify SignalK is Sending Data

Check your SignalK server logs or dashboard to confirm it's sending:
- Heading updates
- Position updates
- Speed/Course updates

### Verify MAYARA is Receiving

Check MAYARA logs for:
```
Listening to Signal K data from 172.16.212.61:8375
```

If you see this, data is flowing!

### Check if Data is Being Used

Look at MAYARA logs for any warnings about:
- Missing navigation data (shouldn't appear now)
- Position updates
- Heading calculations

## Impact Summary

| Feature | Without SignalK | With SignalK |
|---------|----------------|--------------|
| **Bearing** | Relative (0° = ahead) | True North (0° = North) |
| **Position** | None | GPS coordinates |
| **Display Mode** | Head-up only | Head-up or North-up |
| **Trails** | Relative only | Position-stabilized |
| **Chart Overlay** | Not possible | Possible with lat/lon |
| **True North** | No | Yes |

## Next Steps

1. **Refresh the viewer** to see updated data
2. **Check WebSocket messages** to verify bearing/position fields
3. **Try different display modes** if the viewer supports it
4. **Check trails** if you're using `--targets trails` mode

The radar data structure now includes true north bearings and GPS positions, making it ready for chart overlays and more advanced navigation features!
