# TD-to-LED

### Creating RGB in TouchDesigner and Sending Data to Programmable LEDs

---

### Overview
I wanted to design an RGB lighting setup using the visual power of **TouchDesigner** for some interesting and customisable visuals. This was to start to understand how DMX works for installations and AV setups, and work out how that works practically in TouchDesigner. My solution was to use small, **programmable LEDs** connected to a microcomputer that can receive DMX data. Having small LEDs means we can broadcast small amounts of data and use Wifi for a minimal home setup.

The LED strip receives RGB data only, so the DMX stream will need:
- R, G, B values for each pixel, for 60 LEDs.
- A total of 180 DMX channels

### **TouchDesigner Network**
Here is a screenshot of a simple TouchDesigner network that converts an image into the correct DMX format:

![TouchDesigner Network](https://github.com/user-attachments/assets/681eddfc-603d-487a-81ae-863f2a069550)

- Each pixel in the input image corresponds to an LED
- The input image is 60x1, matching the LED srip
- `shuffle1` arranges RGB data into a single sequence: `r1, g1, b1, r2, g2, ...`
- `shuffle2` splits this sequence into **individual DMX channels**. We have to use 2 shuffles to get the data in the correct order.
- `rename1` renames channels to `ch*` (optional, but reflects how DMX data is processed).
- `math1` scales values from TouchDesigner's 0-1 range to DMX's 0-255 range and rounds to integers.

Final output format of DMX:

![DMX Output](https://github.com/user-attachments/assets/b3407530-423b-4b13-873f-f4a1be9314cc)

Now, the data is in the correct DMX format for the LEDs. We use a `dmxout` node to broadcast it to the **microcontroller's local IP address**.

---

## Microcontroller Code (Arduino)
The microcontroller needs to:
- Connect to WiFi (and tell me it's IP address)
- Use an LED library compatible with our hardware
- Receive DMX data via ArtNet

### **Setting up Libraries and initialising code**
```cpp
#include <ArtnetWiFi.h>
#include <WiFi.h>
#include <FastLED.h>

ArtnetWiFi artnet;
uint8_t net = 0;
uint8_t subnet = 0;
uint8_t universe = 0;

#define DATA_PIN 2
#define NUM_LEDS 60
CRGB leds[NUM_LEDS];

const char* ssid = "YOUR_WIFI_NETWORK";
const char* password = "YOUR_WIFI_PASSWORD";
```

### **WiFi Connection Setup**
```cpp
Serial.begin(115200);
WiFi.begin(ssid, password);
delay(1000);
while (WiFi.status() != WL_CONNECTED) {
  delay(1000);
  Serial.println("Connecting to WiFi...");
}
Serial.println("Connected to WiFi");
Serial.print("IP Address: ");
Serial.println(WiFi.localIP());
```

### **LED Setup**
```cpp
FastLED.addLeds<WS2812, DATA_PIN, GRB>(leds, NUM_LEDS);
```

### **Receiving ArtNet DMX Data**
```cpp
artnet.begin();

artnet.subscribeArtDmxUniverse(universe, [](const uint8_t *data, uint16_t size, const ArtDmxMetadata& metadata, const ArtNetRemoteInfo& remote)
{
    for (size_t pixel = 0; pixel < NUM_LEDS; ++pixel)
    {
        size_t idx = pixel * 3;
        leds[pixel].r = data[idx + 0];
        leds[pixel].g = data[idx + 1];
        leds[pixel].b = data[idx + 2];
    }
});
```

### **Main Loop**
```cpp
void loop() {
  artnet.parse();
  FastLED.show();
}
```

---

Now, once the code is running on the microcontroller, and the TouchDesigner network is open, We can now customize any visual we want from TouchDesigner, including using audio-reactivity, and broadcast to LEDs in real time.
This was very satisfying to get working, and the theory of this broadcasting should extend for larger projects as well, for instance to large LED screens.
I was pleased to learn about all parts of the process, from electronics setup, to network protocals, to writing code for a microcontroller.



https://github.com/user-attachments/assets/7585e985-4bd3-4fbf-b242-b5dda76356f5


---

## **Future Improvements**
- WiFi works well, but for larger installations, a direct **Ethernet connection** may be needed.

- The current setup uses a breadboard and taped wires. A **soldered circuit** with a protective shell would make it more permanent.

- Add more LEDs for a richer visual experience

