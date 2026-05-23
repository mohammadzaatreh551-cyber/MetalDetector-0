#include <Wire.h>
#include <ESP8266WebServer.h>

ESP8266WebServer server(80);

void setup() {
  Wire.begin();
  Serial.begin(115200);
  
  // تهيئة الحساس QMC5883L مباشرة عبر السجلات
  Wire.beginTransmission(0x0D);
  Wire.write(0x0B); // Set/Reset register
  Wire.write(0x01);
  Wire.endTransmission();
  
  Wire.beginTransmission(0x0D);
  Wire.write(0x09); // Control register
  Wire.write(0x1D); // Mode: Continuous, 200Hz, 8G, 512OSR
  Wire.endTransmission();

  // إعداد الـ Wi-Fi والـ Server... (يتم إضافته هنا)
}

void getSensorData() {
  Wire.beginTransmission(0x0D);
  Wire.write(0x00); // Data register
  Wire.endTransmission();
  Wire.requestFrom(0x0D, 6);
  
  if(Wire.available() == 6) {
    int x = Wire.read() | (Wire.read() << 8);
    int y = Wire.read() | (Wire.read() << 8);
    int z = Wire.read() | (Wire.read() << 8);
    
    // إرسال البيانات بتنسيق JSON ليقرأها التطبيق
    String json = "{\"x\":" + String(x) + ",\"y\":" + String(y) + ",\"z\":" + String(z) + "}";
    server.send(200, "application/json", json);
  }
}
