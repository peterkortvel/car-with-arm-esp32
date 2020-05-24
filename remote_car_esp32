#include <WiFi.h>
#include <SPIFFS.h>
#include <ESPAsyncWebServer.h>
#include <WebSocketsServer.h>

// Robot
#include <ESP32Servo.h>
Servo myservo3; 
Servo myservo1; 
Servo myservo2;
Servo myservo4;
 
// Constants
const char *ssid = "ESP32-AP";
const char *password =  "LetMeInPlz";
const char *msg_toggle_led = "toggleLED";
const char *msg_get_led = "getLEDState";
const int dns_port = 53;
const int http_port = 80;
const int ws_port = 1337;
const int led_pin = 32;
const int armGround_pin = 27;
const int armDistance_pin = 15;
const int armHeight_pin = 14;
const int pin_speed1 = A0;
const int pin_motor1a = 21;
const int pin_motor1b = A5;
const int pin_motor2a = 13;
const int pin_motor2b = 33;

 
// Globals
AsyncWebServer server(80);
WebSocketsServer webSocket = WebSocketsServer(1337);
char msg_buf[10];
int led_state = 120;
int arm_ground = 0;
 
/***********************************************************
 * Functions
 */
 
// Callback: receiving any WebSocket message
void onWebSocketEvent(uint8_t client_num,
                      WStype_t type,
                      uint8_t * payload,
                      size_t length) {
 
  // Figure out the type of WebSocket event
  switch(type) {
 
    // Client has disconnected
    case WStype_DISCONNECTED:
      Serial.printf("[%u] Disconnected!\n", client_num);
      break;
 
    // New client has connected
    case WStype_CONNECTED:
      {
        IPAddress ip = webSocket.remoteIP(client_num);
        Serial.printf("[%u] Connection from ", client_num);
        Serial.println(ip.toString());
      }
      break;
 
    // Handle text messages from client
    case WStype_TEXT:
      {
        String string_payload = (char *)payload;
        String motor_selector = string_payload.substring(0,4);
        String motor_move_string = string_payload.substring(4);
        int motor_move = motor_move_string.toInt();
        Serial.println(motor_selector);
        Serial.println(motor_move);
        
        
      // Print out raw message
      Serial.printf("[%u] Received text: %s\n", client_num, payload);

 
      // Toggle LED
      if ( motor_selector == "togg" ) {
        led_state = (led_state==120) ? 180 : 120;
        Serial.printf("Toggling LED to %u\n", led_state);
      // Report the state of the LED
      } else if (motor_selector == "gets" ) {
        led_state = myservo1.read();
        sprintf(msg_buf, "%d", led_state);
        Serial.printf("Sending to [%u]: %s\n", client_num, msg_buf);
        webSocket.sendTXT(client_num, msg_buf);
      } else if (motor_selector == "turn") {
        myservo1.write(motor_move);
      } else if (motor_selector == "dist") {
        myservo2.write(motor_move);
      } else if (motor_selector == "heig") {
        myservo4.write(motor_move);
      } else if (motor_selector == "clam") {
        myservo3.write(motor_move);
      } else if (motor_selector == "gogo") {
        motorMove("go");
        delay(400);
        motorMove("stop");
      } else if (motor_selector == "back") {
        motorMove("back");
        delay(400);
        motorMove("stop");
      } else if (motor_selector == "left") {
        motorMove("left");
        delay(100);
        motorMove("stop");
      } else if (motor_selector == "righ") {
        motorMove("right");
        delay(100);
        motorMove("stop");
      }  else {
 
        Serial.println("[%u] Message not recognized");
      }
      break;
      }
    // For everything else: do nothing
    case WStype_BIN:
    case WStype_ERROR:
    case WStype_FRAGMENT_TEXT_START:
    case WStype_FRAGMENT_BIN_START:
    case WStype_FRAGMENT:
    case WStype_FRAGMENT_FIN:
    default:
      break;
  }
}
 
// Callback: send homepage
void onIndexRequest(AsyncWebServerRequest *request) {
  IPAddress remote_ip = request->client()->remoteIP();
  Serial.println("[" + remote_ip.toString() +
                  "] HTTP GET request of " + request->url());
  request->send(SPIFFS, "/index.html", "text/html");
}
 
// Callback: send style sheet
void onCSSRequest(AsyncWebServerRequest *request) {
  IPAddress remote_ip = request->client()->remoteIP();
  Serial.println("[" + remote_ip.toString() +
                  "] HTTP GET request of " + request->url());
  request->send(SPIFFS, "/style.css", "text/css");
}
 
// Callback: send 404 if requested file does not exist
void onPageNotFound(AsyncWebServerRequest *request) {
  IPAddress remote_ip = request->client()->remoteIP();
  Serial.println("[" + remote_ip.toString() +
                  "] HTTP GET request of " + request->url());
  request->send(404, "text/plain", "Not found");
}
 
/***********************************************************
 * Main
 */
 
void setup() {
  myservo3.attach(led_pin);
  myservo1.attach(armGround_pin);
  myservo2.attach(armDistance_pin);
  myservo4.attach(armHeight_pin);

  pinMode(pin_speed1,OUTPUT);
  pinMode(pin_motor1a,OUTPUT);
  pinMode(pin_motor1b,OUTPUT);
  pinMode(pin_motor2a,OUTPUT);
  pinMode(pin_motor2b,OUTPUT);
  
  // Init LED and turn off
  pinMode(led_pin, OUTPUT);
  digitalWrite(led_pin, LOW);
 
  // Start Serial port
  Serial.begin(115200);
 
  // Make sure we can read the file system
  if( !SPIFFS.begin()){
    Serial.println("Error mounting SPIFFS");
    while(1);
  }
 
  // Start access point
  WiFi.softAP(ssid, password);
 
  // Print our IP address
  Serial.println();
  Serial.println("AP running");
  Serial.print("My IP address: ");
  Serial.println(WiFi.softAPIP());
 
  // On HTTP request for root, provide index.html file
  server.on("/", HTTP_GET, onIndexRequest);
 
  // On HTTP request for style sheet, provide style.css
  server.on("/style.css", HTTP_GET, onCSSRequest);
 
  // Handle requests for pages that do not exist
  server.onNotFound(onPageNotFound);
 
  // Start web server
  server.begin();
 
  // Start WebSocket server and assign callback
  webSocket.begin();
  webSocket.onEvent(onWebSocketEvent);
  
}
 
void loop() {
  
  // Look for and handle WebSocket data
  webSocket.loop();
}

void motorMove(String move) {
if (move.equals("go"))    
    {
      digitalWrite(pin_speed1,HIGH);
      digitalWrite(pin_motor1a,HIGH);
      digitalWrite(pin_motor1b,LOW);
      digitalWrite(pin_motor2a,HIGH);
      digitalWrite(pin_motor2b,LOW);
    }
else if (move.equals("back"))    
    { 
      digitalWrite(pin_speed1,HIGH);
      digitalWrite(pin_motor1a,LOW);
      digitalWrite(pin_motor1b,HIGH);
      digitalWrite(pin_motor2a,LOW);
      digitalWrite(pin_motor2b,HIGH);
    }
else if (move.equals("left"))    
    {
      digitalWrite(pin_speed1,HIGH);
      digitalWrite(pin_motor1a,LOW);
      digitalWrite(pin_motor1b,HIGH);
      digitalWrite(pin_motor2a,HIGH);
      digitalWrite(pin_motor2b,LOW);
    }
else if (move.equals("right"))    
    {
      digitalWrite(pin_speed1,HIGH);
      digitalWrite(pin_motor1a,HIGH);
      digitalWrite(pin_motor1b,LOW);
      digitalWrite(pin_motor2a,LOW);
      digitalWrite(pin_motor2b,HIGH);
    }
else if (move.equals("stop"))    
    {
      digitalWrite(pin_speed1,LOW);
    }
else
    {
  // nothing
    }

}
