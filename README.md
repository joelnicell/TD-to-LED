# TD-to-LED

### Creating RGB in TouchDesigner and Sending Data to Programmable LEDs

---

## Overview
I wanted to design an RGB lighting setup using **TouchDesigner** for full customization and display it on hardware. My solution was to use **programmable LEDs** connected to a **microcomputer** that can receive DMX data. Since TouchDesigner can output visuals as DMX, I wanted to broadcast this data to the microcontroller.

Once I understood the electronics, I wrote code for the microcontroller to **receive DMX data** and translate it into **RGB values** for the LEDs. The code for this can be found in: `_____`.

Next, I needed to generate the correct **DMX format** from TouchDesigner. Below is the network setup for achieving this.

---

## DMX Format for LEDs
The LED strip receives RGB data only, so the **DMX stream** consists of:
- **R, G, B values** for each pixel
- **60 LEDs** → **180 DMX channels** (3 per LED)

Here is a screenshot of a simple TouchDesigner network that converts an image into the correct DMX format:

![TouchDesigner Network](https://github.com/user-attachments/assets/681eddfc-603d-487a-81ae-863f2a069550)

### **Breakdown of the TouchDesigner Network**
- **Each pixel** in the input image corresponds to an **LED**.
- `shuffle1` arranges RGB data into a single sequence: `r1, g1, b1, r2, g2, ...`
- `shuffle2` splits this sequence into **individual DMX channels**.
- `rename1` renames channels to `ch*` (optional for DMX compliance).
- `math1` scales values from **TouchDesigner's 0-1 range** to **DMX's 0-255 range** and rounds to integers.

Final output format:

![DMX Output](https://github.com/user-attachments/assets/b3407530-423b-4b13-873f-f4a1be9314cc)

Now, the data is in the correct DMX format for the LEDs. We use `dmxout` to broadcast it to the **microcontroller's local IP address**.

---

## Microcontroller Code (Arduino)
The microcontroller needs to:
- **Connect to WiFi**
- **Use an LED library** compatible with our hardware
- **Receive DMX data via ArtNet**

### **Required Libraries**
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

// Subscribe to DMX universe
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

## **Final Results**
With everything set up, the project works! We can now:
- **Customize LED visuals in real-time** from TouchDesigner.
- **Use audio-reactivity** to create dynamic lighting effects.
- **Stream data wirelessly** to our LEDs.

---

## **Future Improvements**
- **Network Speed:** WiFi works well, but for larger installations, a direct **Ethernet connection** may be needed.

- **Hardware Durability:** The current setup uses a **breadboard** and **taped wires**. A **soldered circuit** with a protective shell would make it more permanent.

---

- **This project bridges TouchDesigner with LED hardware for real-time generative visuals!**

