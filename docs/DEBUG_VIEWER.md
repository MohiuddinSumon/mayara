# Debugging Viewer Issues

If you see data in the terminal but not in `viewer.html`, follow these steps:

## Step 1: Check if Radar is Detected

Open your browser and go to:
```
http://localhost:6502/v1/api/radars
```

Or use curl:
```bash
curl http://localhost:6502/v1/api/radars
```

**Expected output**: A JSON object with radar entries like:
```json
{
  "radar-1": {
    "id": "radar-1",
    "name": "Radar Name",
    "spokesPerRevolution": 1024,
    "maxSpokeLen": 2048,
    "streamUrl": "ws://localhost:6502/v1/api/spokes/radar-1",
    "controlUrl": "ws://localhost:6502/v1/api/control/radar-1",
    ...
  }
}
```

**If empty `{}`**: The radar is not detected yet. This can happen if:
- Packets are being received but radar detection hasn't completed
- The radar needs to send specific packets to be detected
- Wait a bit longer and check again

## Step 2: Check Browser Console

1. Open `viewer.html` in your browser: `http://localhost:6502/viewer.html`
2. Open Developer Tools (F12 or Right-click → Inspect)
3. Go to the Console tab
4. Look for errors or messages

**Common issues**:
- `WebSocket connection failed`: Check the radar ID in URL
- `radar-1 is undefined`: Radar not detected, check Step 1
- `Failed to load proto file`: Check network tab for 404 errors

## Step 3: Use Correct Radar ID

The viewer needs an `id` parameter in the URL. If you see a radar in the API:

1. **Option A**: Use the main page to get links:
   ```
   http://localhost:6502/
   ```
   This will show detected radars with clickable links.

2. **Option B**: Manually add the ID:
   ```
   http://localhost:6502/viewer.html?id=radar-1
   ```
   Replace `radar-1` with the actual ID from the API response.

## Step 4: Verify Radar is Active

The API only returns "active" radars. A radar becomes active when:
- It's detected by the locator
- It starts receiving data packets
- It has completed initial setup

If you see data in terminal but radar isn't in API:
- Wait a few seconds for detection to complete
- Check server logs for radar detection messages
- Verify the brand matches (`--brand navico` for Navico radars)

## Step 5: Check WebSocket Connection

Once the viewer loads, check the Network tab in Developer Tools:
1. Filter by "WS" (WebSocket)
2. Look for connection to `/v1/api/spokes/radar-X`
3. Status should be "101 Switching Protocols"
4. You should see binary messages being received

## Common Fixes

### Fix 1: Access via Main Page
Instead of going directly to `viewer.html`, go to:
```
http://localhost:6502/
```
This page will show all detected radars with proper links.

### Fix 2: Wait for Detection
Radar detection can take a few seconds. Keep the API page open and refresh until you see a radar.

### Fix 3: Check Port
Make sure you're using the correct port:
- Default: `6502`
- Docker demo: `3001`
- Check what port your server is actually using

### Fix 4: Check Server Logs
Look at the terminal where mayara-server is running. You should see:
```
Found radar: key '...' id 1 name '...'
```
If you don't see this, the radar isn't being detected.

## Still Not Working?

1. **Check browser console** for JavaScript errors
2. **Check Network tab** for failed requests
3. **Verify server is running** on the expected port
4. **Check server logs** for radar detection messages
5. **Try a different browser** to rule out browser-specific issues
