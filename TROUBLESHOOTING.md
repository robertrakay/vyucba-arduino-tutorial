# Troubleshooting Guide

## Problems Encountered & Solutions

### 1. WiFi Connection Issues

**Problem**: Arduino GIGA cannot connect to 5 GHz WiFi network

**Root Cause**: The Murata 1DX WiFi module only supports 2.4 GHz (802.11 b/g/n)

**Solution**: 
- Use 2.4 GHz WiFi network only
- Check router settings for separate 2.4 GHz SSID
- Update `config.h` with correct 2.4 GHz network name

**Code Fix**:
```cpp
// config.h
const char* WIFI_SSID = "YOUR_WIFI_2.4GHz";  // NOT "YOUR_WIFI_5G"
```

---

### 2. Compilation Error: Unterminated #ifndef

**Problem**: Compiler error about unterminated header guards

**Root Cause**: Missing `#endif` in header files (especially `html_pages.h`)

**Solution**: 
- Ensure every `#ifndef` has matching `#endif`
- Use `F()` macro to save RAM when storing HTML strings

**Code Fix**:
```cpp
#ifndef HTML_PAGES_H
#define HTML_PAGES_H

// ... your code ...

#endif  // <-- Don't forget this!
```

---

### 3. LEDs Not Responding to Control

**Problem**: Web interface shows LED state changes, but physical LEDs don't light up

**Root Cause**: 
- RGB LED pins not correctly defined
- Arduino GIGA uses pins 86, 87, 88 (not standard LED_BUILTIN)
- LEDs are **active-LOW** (LOW = ON, HIGH = OFF)

**Solution**:
```cpp
// config.h - Explicit pin definitions
#ifndef LED_RED
  #define LED_RED 86
#endif
#ifndef LED_GREEN
  #define LED_GREEN 87
#endif
#ifndef LED_BLUE
  #define LED_BLUE 88
#endif

// led_control.h - Active-LOW logic
digitalWrite(LED_RED, !state);  // Invert: state=1 → LOW (ON)
```

---

### 4. String Reference Binding Error

**Problem**: `cannot bind non-const lvalue reference of type 'arduino::String&'`

**Root Cause**: WebSocket `broadcastTXT()` expects non-const String reference, but we passed temporary String

**Solution**: Create String variable first, then pass it
```cpp
// WRONG:
webSocket.broadcastTXT(getLEDStateJSON());

// CORRECT:
String jsonData = getLEDStateJSON();
webSocket.broadcastTXT(jsonData);
```

---

### 5. Serial.printf() Not Available

**Problem**: Compilation error on `Serial.printf()`

**Root Cause**: Arduino GIGA (Mbed OS) doesn't support `printf()` on Serial

**Solution**: Use `Serial.print()` with multiple calls
```cpp
// WRONG:
Serial.printf("[%u] Connected\n", clientNum);

// CORRECT:
Serial.print("[");
Serial.print(clientNum);
Serial.println("] Connected");
```

---

### 6. MATLAB Connection Issues

**Problem**: Cannot connect MATLAB Online to Arduino via UDP

**Root Cause**: MATLAB Online runs in cloud; Arduino is on local network

**Solution**: Use MQTT broker or MATLAB Desktop (local)
- **MATLAB Online**: Use MQTT cloud broker (HiveMQ, etc.)
- **MATLAB Desktop**: Use TCP/IP or UDP directly

---

### 7. WebSocket Disconnecting Repeatedly

**Problem**: WebSocket connection drops after a few seconds

**Root Cause**: 
- `webSocket.loop()` not called frequently enough in main loop
- Blocking code preventing WebSocket processing

**Solution**:
```cpp
void loop() {
  handleHTTPRequests();  // Keep short
  webSocket.loop();       // Call every iteration
  // Avoid delay() or long blocking operations
}
```

---

### 8. Login Page Shows "Invalid Credentials" Even with Correct Password

**Problem**: POST body parsing fails

**Root Cause**: POST data not fully received before parsing

**Solution**: Add delay to ensure full body received
```cpp
delay(50);  // Give client time to send body
while(client.available()) {
  body += (char)client.read();
}
```

---

### 9. History Log Not Updating

**Problem**: Activity log shows old or missing entries

**Root Cause**: Circular buffer index calculation error

**Solution**: Use modulo arithmetic correctly
```cpp
historyIndex = (historyIndex + 1) % HISTORY_SIZE;
```

Display in reverse:
```cpp
for(int i = historyCount - 1; i >= 0; i--) {
  int idx = (historyIndex - historyCount + i + HISTORY_SIZE) % HISTORY_SIZE;
  // ...
}
```

---

### 10. mDNS Not Working

**Problem**: Cannot access `http://giga-led.local`

**Root Cause**: Arduino GIGA (Mbed OS) has different mDNS implementation than ESP32

**Solution**: Use IP address directly for now
- Feature in development for future releases
- Alternative: Set static IP in router DHCP settings

---

## Debug Tips

### Enable Verbose Serial Output

Add to `setup()`:
```cpp
Serial.begin(115200);
while (!Serial) { delay(10); }  // Wait for Serial Monitor
Serial.println("\n=== DEBUG MODE ===");
```

### Monitor WiFi Connection

```cpp
Serial.print("WiFi status: ");
Serial.println(WiFi.status());  // 3 = WL_CONNECTED
Serial.print("Signal strength: ");
Serial.print(WiFi.RSSI());
Serial.println(" dBm");
```

### Check Memory Usage

```cpp
extern "C" char* sbrk(int incr);
int freeMemory() {
  char top;
  return &top - reinterpret_cast<char*>(sbrk(0));
}

Serial.print("Free RAM: ");
Serial.println(freeMemory());
```

---

## Platform-Specific Notes

### Windows
- Use Bonjour Print Services for `.local` hostname resolution (if mDNS implemented)
- Check firewall isn't blocking ports 80/81

### macOS
- Built-in Bonjour support for `.local` hostnames
- Safari may cache old credentials; use Chrome/Firefox for testing

### Linux
- Install `avahi-daemon` for `.local` hostname resolution
- May need to add user to `dialout` group for serial port access:
  ```bash
  sudo usermod -a -G dialout $USER
  ```

---

## Getting Help

If you encounter issues not listed here:

1. **Check Serial Monitor** output for error messages
2. **Test basic connectivity**: Can you ping the Arduino's IP?
3. **Verify libraries**: Ensure WebSocketsServer and ArduinoJson are installed
4. **Open GitHub Issue**: Provide Serial Monitor output and code changes

---

*Last updated: April 26, 2026*
