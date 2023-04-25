# ESP32_教材
### 教學項目_Wifi & Bluetooth
#### Bluetooth
只有一個實驗
>
因為APP必須分成Andriod and IOS各一個，故用Blynk執行即可
>

#### Wifi
共有四個實驗
1. Connect to Wifi
2. Control LED via Wifi
3. ESP32 Use ChatGPT API
4. 將ESP32上的資料傳到雲端上

ESP32_Wifi連線:
用wifi.h程式庫會有4種wifi模式可以用:
語法 | Wifi模式 | 功能 
|---|---|---|
Wifi.mode(WIFI_AP); | Access Point(AP) | ESP32可以讓其他設備透過wifi接入(AP 模式是指由 ESP 模組擔任無線基地台，連上它可形成封閉的區域網路)
WiFi.mode(WIFI_STA); | Station(STA) | 無線終端模式(STA 模式則是 ESP 用 SSID 及密碼連上其他基地台取得 IP，之後便可上網，也能跑伺服器對區域網路提供服務)
WiFi.mode(WIFI_AP_STA); | AP+STA | 將EPS32設置兩個模式並存


##### 1. 連上wifi:

Get WiFi Connection Strength code
```
/*
  Complete details at https://RandomNerdTutorials.com/esp32-useful-wi-fi-functions-arduino/
*/

#include <WiFi.h>

// Replace with your network credentials (STATION)
const char* ssid = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";

void initWiFi() {
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('No Connection');
    delay(1000);
  }
  Serial.println(WiFi.localIP());
}

void setup() {
  Serial.begin(115200); //開啟裝置管理員確認usb與資料傳輸速度與此相同
  initWiFi();
  Serial.print("RRSI: ");
  Serial.println(WiFi.RSSI());
}

void loop() {
  // put your main code here, to run repeatedly:
}
```

##### 2. Control LED via Wifi


```
#include <WiFi.h>

// Replace with your network credentials
const char* ssidAP     = "****"; //Esp32s基地台名稱(
const char* passwordAP = "********";//Esp32s基地台密碼(至少要8碼)

// Set web server port number to 80
WiFiServer server(80);

// Variable to store the HTTP request
String header;

// Auxiliar variables to store the current output state
String output10State = "off";
String output12State = "off";

// Assign output variables to GPIO pins
const int output10 = 10;
const int output12 = 12;

void setup() {
  Serial.begin(115200);
  // Initialize the output variables as outputs
  pinMode(output10, OUTPUT);
  pinMode(output12, OUTPUT);
  // Set outputs to LOW
  digitalWrite(output10, LOW);
  digitalWrite(output12, LOW);

  // Connect to Wi-Fi network with SSID and password
  Serial.print("Setting AP (Access Point)…");
  // Remove the password parameter, if you want the AP (Access Point) to be open
  WiFi.softAP(ssidAP, passwordAP);
  delay(500);
  IPAddress IP = WiFi.softAPIP();
  Serial.print("AP IP address: ");
  Serial.println(IP);
  
  server.begin();
}

void loop(){
  WiFiClient client = server.available();   // Listen for incoming clients

  if (client) {                             // If a new client connects,
    Serial.println("New Client.");          // print a message out in the serial port
    String currentLine = "";                // make a String to hold incoming data from the client
    while (client.connected()) {            // loop while the client's connected
      if (client.available()) {             // if there's bytes to read from the client,
        char c = client.read();             // read a byte, then
        Serial.write(c);                    // print it out the serial monitor
        header += c;
        if (c == '\n') {                    // if the byte is a newline character
          // if the current line is blank, you got two newline characters in a row.
          // that's the end of the client HTTP request, so send a response:
          if (currentLine.length() == 0) {
            // HTTP headers always start with a response code (e.g. HTTP/1.1 200 OK)
            // and a content-type so the client knows what's coming, then a blank line:
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println("Connection: close");
            client.println();
            
            // turns the GPIOs on and off
            if (header.indexOf("GET /10/on") >= 0) {
              Serial.println("GPIO 10 on");
              output10State = "on";
              digitalWrite(output10, HIGH);
            } else if (header.indexOf("GET /10/off") >= 0) {
              Serial.println("GPIO 10 off");
              output10State = "off";
              digitalWrite(output10, LOW);
            } else if (header.indexOf("GET /12/on") >= 0) {
              Serial.println("GPIO 12 on");
              output12State = "on";
              digitalWrite(output12, HIGH);
            } else if (header.indexOf("GET /12/off") >= 0) {
              Serial.println("GPIO 12 off");
              output12State = "off";
              digitalWrite(output12, LOW);
            }
            
            // Display the HTML web page
            client.println("<!DOCTYPE html><html>");
            client.println("<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">");
            client.println("<link rel=\"icon\" href=\"data:,\">");
            // CSS to style the on/off buttons 
            // Feel free to change the background-color and font-size attributes to fit your preferences
            client.println("<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}");
            client.println(".button { background-color: #4CAF50; border: none; color: white; padding: 16px 40px;");
            client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
            client.println(".button2 {background-color: #555555;}</style></head>");
            
            // Web Page Heading
            client.println("<body><h1>ESP32 Web Server</h1>");
            
            // Display current state, and ON/OFF buttons for GPIO 10  
            client.println("<p>GPIO 10 - State " + output10State + "</p>");
            // If the output0State is off, it displays the ON button       
            if (output10State=="off") {
              client.println("<p><a href=\"/10/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/10/off\"><button class=\"button button2\">OFF</button></a></p>");
            } 
               
            // Display current state, and ON/OFF buttons for GPIO 12  
            client.println("<p>GPIO 12 - State " + output12State + "</p>");
            // If the output2State is off, it displays the ON button       
            if (output12State=="off") {
              client.println("<p><a href=\"/12/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/12/off\"><button class=\"button button2\">OFF</button></a></p>");
            }
            client.println("</body></html>");
            
            // The HTTP response ends with another blank line
            client.println();
            // Break out of the while loop
            break;
          } else { // if you got a newline, then clear currentLine
            currentLine = "";
          }
        } else if (c != '\r') {  // if you got anything else but a carriage return character,
          currentLine += c;      // add it to the end of the currentLine
        }
      }
    }
    // Clear the header variable
    header = "";
    // Close the connection
    client.stop();
    Serial.println("Client disconnected.");
    Serial.println("");
  }
}
```

##### 3. ESP32 Use ChatGPT API
```
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

const char* WIFI_SSID = "Pixel 4a";
const char* WIFI_PASSWORD = "QWERtyuiop000";

// Replace with your OpenAI API key
const char* apiKey = "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";

void setup() {
  // Initialize Serial
  Serial.begin(115200);

  // Connect to Wi-Fi network
  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
      Serial.print('.');
      delay(1000);
  }
  Serial.println(WiFi.localIP());
  
  // Send request to OpenAI API
  String inputText = "請告訴我今天中央大學的氣溫是多少??";  
  Serial.println(inputText);
  String apiUrl = "https://api.openai.com/v1/completions";
  String payload = "{\"prompt\":\"" + inputText + "\",\"max_tokens\":100, \"model\": \"text-davinci-003\"}";

  HTTPClient http;
  http.begin(apiUrl);
  http.addHeader("Content-Type", "application/json");
  http.addHeader("Authorization", "Bearer " + String(apiKey));
  
  int httpResponseCode = http.POST(payload);
  if (httpResponseCode == 200) {
    String response = http.getString();
  
    // Parse JSON response
    DynamicJsonDocument jsonDoc(1024);
    deserializeJson(jsonDoc, response);
    String outputText = jsonDoc["choices"][0]["text"];
    Serial.println(outputText);
  } else {
    Serial.printf("Error %i \n", httpResponseCode);
  }
}

void loop() {
  // do nothing
}
```


OpenAI API URL: https://platform.openai.com/account/api-keys
##### 4.
https://youyouyou.pixnet.net/blog/post/119623728




參考資料:
- https://randomnerdtutorials.com/esp32-useful-wi-fi-functions-arduino/
- https://peterpowerfullife.com/blog/tech-rssi/
- https://ithelp.ithome.com.tw/articles/10262962
