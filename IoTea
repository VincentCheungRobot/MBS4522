#include <PubSubClient.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <SPI.h>
#include <MFRC522.h>
#include <Adafruit_PWMServoDriver.h>


//OLED Setting
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);
//The most suitable pins for I2C communication in the ESP32 are GPIO 22 (SCL) and GPIO 21 (SDA).

#define SS_PIN 5
#define RST_PIN 27
MFRC522 rfid(SS_PIN, RST_PIN);
// Sr No	RFID RC522 Pins	ESP32 Pins
// 1	VCC	+3.3V
// 2	RST	D0
// 3	GND	GND
// 4	MISO	19
// 5	MOSI	23
// 6	SCK	18
// 7	SS/SDA	5

// Servo Setting

Adafruit_PWMServoDriver pwm = Adafruit_PWMServoDriver(PCA9685_I2C_ADDRESS);
// Depending on your servo make, the pulse width min and max may vary, you 
// want these to be as small/large as possible without hitting the hard stop
// for max range. You'll have to tweak them as necessary to match the servos you
// have!
#define SERVOMIN 150 // Minimum value
#define SERVOMAX 300 // Maximum value
#define SERVO_FREQ 50 // Analog servos run at ~50 Hz updates
// WiFi Setting

// const char* ssid = "ukw";
// const char* password = "Q25tU0KaA";
// const char* ssid = "TP-Link_E944";
// const char* password = "SeeWhat9?";
const char* ssid = "Linksys03688";
const char* password = "ivelwl2022";
const char* ntpServer = "pool.ntp.org";
const char* mqtt_server = "test.mosquitto.org";
const long gmtOffset_sec = 28800;
const int daylightOffset_sec = 0;
struct tm timeinfo;

WiFiClient espClient;
PubSubClient client(espClient);

const int maxParkingSpaces = 5;
unsigned long previousMillis = 0;
unsigned long currentMillis ;
const long interval = 5000;
int remainingSpaces = maxParkingSpaces;
bool isFull = false;
char* EntryState ;
#include <set>
std::set<String> parkedCars;

//Use 74HC165 to monitor the occupancy of 5 parking spaces with 4 pins.

const int data_pin = 36; // Connect Pin 11 to SER_OUT (serial data out)
const int shld_pin = 25; // Connect Pin 8 to SH/!LD (shift or active low load)
const int clk_pin = 33; // Connect Pin 12 to CLK (the clock that times the shifting)
const int ce_pin = 32; // Connect Pin 9 to !CE (clock enable, active low)

byte incoming; // Variable to store the 8 values loaded from the shift register

// This function is executed when some device publishes a message to a topic that your ESP8266 is subscribed to
// Change the function below to add logic to your program, so when a device publishes a message to a topic that 
// your ESP8266 is subscribed you can actually do something
void callback(String topic, byte* message, unsigned int length) {
  Serial.print("Message arrived on topic: ");
  Serial.print(topic);
  Serial.print(". Message: ");
  String messageTemp;
  
  for (int i = 0; i < length; i++) {
    Serial.print((char)message[i]);
    messageTemp += (char)message[i];
  }
  Serial.println();
 
  // Feel free to add more if statements to control more GPIOs with MQTT

  Serial.println();
}

// This functions reconnects your ESP8266 to your MQTT broker
// Change the function below if you want to subscribe to more topics with your ESP8266 
void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Attempt to connect
    /*
     YOU MIGHT NEED TO CHANGE THIS LINE, IF YOU'RE HAVING PROBLEMS WITH MQTT MULTIPLE CONNECTIONS
     To change the ESP device ID, you will have to give a new name to the ESP8266.
     Here's how it looks:
       if (client.connect("ESP8266Client")) {
     You can do it like this:
       if (client.connect("ESP1_Office")) {
     Then, for the other ESP:
       if (client.connect("ESP2_Garage")) {
      That should solve your MQTT multiple connections problem
    */
    if (client.connect("ESP32Client_CNM")) {
      Serial.println("connected");  
      // Subscribe or resubscribe to a topic
      // You can subscribe to more topics (to control more LEDs in this example)
      
      // client.subscribe("AIR2324_Temp");
      // client.subscribe("AIR2324_humid");
      // client.subscribe("AIR2324_brightness");
      // client.subscribe("AIR2324_soil_moisture");
      // client.subscribe("AIR2324_LEDStatus");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

// This code is intended to trigger the shift register to grab values from it's A-H inputs
byte read_shift_regs()
{
  byte the_shifted = 0;  // An 8 bit number to carry each bit value of A-H

  // Trigger loading the state of the A-H data lines into the shift register
  digitalWrite(shld_pin, LOW);
  delayMicroseconds(5); // Requires a delay here according to the datasheet timing diagram
  digitalWrite(shld_pin, HIGH);
  delayMicroseconds(5);

  // Required initial states of these two pins according to the datasheet timing diagram
  pinMode(clk_pin, OUTPUT);
  pinMode(data_pin, INPUT);  
  digitalWrite(clk_pin, HIGH);
  digitalWrite(ce_pin, LOW); // Enable the clock

  // Get the A-H values
  the_shifted = shiftIn(data_pin, clk_pin, MSBFIRST);
  digitalWrite(ce_pin, HIGH); // Disable the clock

  return the_shifted;

}

// A function that prints all the 1's and 0's of a byte, so 8 bits +or- 2
void print_byte(byte val)
{
    byte i;
    Serial.print(val >> 0 & 1,BIN);
    client.publish("AIR2324_ParkingSpaceSituation1", String(val >> 0 & 1,BIN).c_str()); 
    client.publish("AIR2324_ParkingSpaceSituation2", String(val >> 1 & 1,BIN).c_str()); 
    client.publish("AIR2324_ParkingSpaceSituation3", String(val >> 2 & 1,BIN).c_str()); 
    client.publish("AIR2324_ParkingSpaceSituation4", String(val >> 3 & 1,BIN).c_str()); 
    client.publish("AIR2324_ParkingSpaceSituation5", String(val >> 4 & 1,BIN).c_str()); 
}

void setup() {
  Serial.begin(115200);
  // Wire.begin();
  pwm.begin();
  SPI.begin();
  pwm.setPWMFreq(SERVO_FREQ);  // Analog servos run at ~50 Hz updates
  rfid.PCD_Init();
//Connect wifi to get real time.
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi connected.Your IP is:");
  Serial.println(WiFi.localIP());
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);

  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);

  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(WHITE);
  display.setCursor(0, 0);
  display.println("Wiring successful");
  display.println();
  display.println();
  display.println();
    if(!getLocalTime(&timeinfo)){
    display.println("Failed to obtain time");
  }else{
    display.println();
    display.println(&timeinfo, "%A, %B %d %Y %H:%M:%S");
  }
  display.display();
  delay(666);
  display.clearDisplay();
  
  // Initialize each digital pin to either output or input
  // We are commanding the shift register with each pin with the exception of the serial
  // data we get back on the data_pin line.
  pinMode(shld_pin, OUTPUT);
  pinMode(ce_pin, OUTPUT);
  pinMode(clk_pin, OUTPUT);
  pinMode(data_pin, INPUT);

  // Required initial states of these two pins according to the datasheet timing diagram
  digitalWrite(clk_pin, HIGH);
  digitalWrite(shld_pin, HIGH);
}


void printLocalTime(){
  if(!getLocalTime(&timeinfo)){
    Serial.println("Failed to obtain time");
    return;
  }
  Serial.println(&timeinfo, "%A, %B %d %Y %H:%M:%S");//參考C++的strftime語法就會知道為甚麼這樣寫
  Serial.print("Day of week: ");
  Serial.println(&timeinfo, "%A");
  Serial.print("Month: ");
  Serial.println(&timeinfo, "%B");
  Serial.print("Day of Month: ");
  Serial.println(&timeinfo, "%d");
  Serial.print("Year: ");
  Serial.println(&timeinfo, "%Y");
  Serial.print("Hour: ");
  Serial.println(&timeinfo, "%H");
  Serial.print("Hour (12 hour format): ");
  Serial.println(&timeinfo, "%I");
  Serial.print("Minute: ");
  Serial.println(&timeinfo, "%M");
  Serial.print("Second: ");
  Serial.println(&timeinfo, "%S");
}

void loop() {

    if (!client.connected()) {
    reconnect();
  }
  if(!client.loop()){    
    client.connect("ESP32Client_CNM");
}

  client.publish("AIR2324_ParkingSpaceSituation6", String(remainingSpaces).c_str()); 
  client.publish("AIR2324_ParkingSpaceSituation", String(isFull).c_str()); 

  //record current time
  currentMillis = millis();
	
  incoming = read_shift_regs(); // Read the shift register, it likes that

  // Print out the values being read from the shift register

  print_byte(incoming); // Print every 1 and 0 that correlates with B through F
  //Serial.println(incoming,BIN); // This way works too but leaves out the leading zeros

    if (currentMillis - previousMillis >= interval) {
    getLocalTime(&timeinfo);
    display.setCursor(0, 0);
  if (isFull) {
    display.println("FULL");
  }else{
    display.println();
  }
    display.println();
    display.println();
    display.println();
    display.println();
    display.println(&timeinfo, "%A, %B %d %Y %H:%M:%S");
    display.display();
    delay(100);
    display.clearDisplay();
    }

  if (!rfid.PICC_IsNewCardPresent() || !rfid.PICC_ReadCardSerial()) {
    return;
  }else{

// Wire.endTransmission();

  //Open the gate.
  for(int pulse=SERVOMAX; pulse>=SERVOMIN; pulse--) {   
    pwm.setPWM(7, 0, pulse);
    Serial.println(pulse);
    delay(10);
  }
  delay(2000);

  //Close the gate.
  for(int pulse=SERVOMIN; pulse<=SERVOMAX; pulse++) {   
    pwm.setPWM(7, 0, pulse);
    Serial.println(pulse);
    delay(10);
  }

  delay(2000);

// Wire.endTransmission();
// Wire.beginTransmission(0x3C); // Oled
  // Wire.endTransmission();
  // delay(100);


  // display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  // display.clearDisplay();
  // display.setTextSize(1);
  // display.setTextColor(WHITE);
  // display.setCursor(0, 0);

    currentMillis = millis();
    //Refresh the last time the card was taken,allow users to see when they enter or leave.
    previousMillis = currentMillis;
  }

  String cardID = "";
  for (byte i = 0; i < rfid.uid.size; i++) {
    cardID += String(rfid.uid.uidByte[i], HEX);
  }

  if (parkedCars.find(cardID) != parkedCars.end()) {
    parkedCars.erase(cardID);
    if (isFull) {
      isFull = false;
    }
    EntryState = "Leave Time:";
    remainingSpaces++;
  } else if (!isFull) {
    parkedCars.insert(cardID);
    EntryState = "Entry Time:";
    remainingSpaces--;

    if (remainingSpaces == 0) {
      isFull = true;
    }
  }

    display.setCursor(0, 0);


  if (isFull) {
    display.println("FULL");
    display.println();
    display.println();
    display.print("Card ID: ");
    display.println(cardID);
    display.println(EntryState);
    display.println(&timeinfo, "%A, %B %d %Y %H:%M:%S");
  } else {
    display.println("Spaces left: ");
    display.println(remainingSpaces);
    display.println();
    display.print("Card ID: ");
    display.println(cardID);
    display.println(EntryState);
    display.println(&timeinfo, "%A, %B %d %Y %H:%M:%S");
  }

  display.display();
  delay(200);
  display.clearDisplay();

  Serial.println(EntryState);
  printLocalTime();
  Serial.print("Card ID: ");
  Serial.println(cardID);
 
  rfid.PICC_HaltA();
  rfid.PCD_StopCrypto1();
}
