#include <TinyGPSPlus.h>
#include <SoftwareSerial.h>
#include <ESP8266WiFi.h>          //https://github.com/esp8266/Arduino 
//needed for library
#include <DNSServer.h>
#include <ESP8266WebServer.h>
#include <WiFiManager.h>          //https://github.com/tzapu/WiFiManager
#include <WiFiClientSecure.h>

/*
   This sample sketch demonstrates the normal use of a TinyGPS++ (TinyGPSPlus) object.
   It requires the use of SoftwareSerial, and assumes that you have a
   4800-baud serial GPS device hooked up on pins 4(rx) and 3(tx). Thank you Mikal for the library.
*/
static const int RXPin = 0, TXPin = 2; //change if you use different pins
static const uint32_t GPSBaud = 9600;

//Button
int button = 5;
int led = 16;
//Device ID
String device = "abhinavGPS"; //Change this
String tim  = "";

// The TinyGPS++ object
TinyGPSPlus gps;

// The serial connection to the GPS device
SoftwareSerial ss(RXPin, TXPin);
//SoftwareSerial ss(4, 5); // The serial connection to the GPS device
//Google Server Setup Section
const char* host = "script.google.com";

const int httpsPort = 443;
WiFiClientSecure client;
const char* fingerprint = "46 B2 C3 44 9C 59 09 8B 01 B6 F8 BD 4C FB 00 74 91 2F EF F6";
String GAS_ID = "AKfycbxMMwl7rGuVgqi_a7bl1gyVL0UlVbTkqGB5ACEnmWPj_ZdHMVWG6P-PLoFvUif6uEyZ";  // Replace by your Google App S service id

float latitude , longitude;
int year , month , date, hour , minute , second;
String date_str , time_str , lat_str , lng_str;
int pm;



WiFiServer server(80);

//Set up to send to GSheets
String readString;

void setup()
{
 
  Serial.begin(115200);
  ss.begin(GPSBaud);

  pinMode(button, INPUT);
  pinMode(led,OUTPUT);
  WiFiManager wifiManager;
  wifiManager.setTimeout(180);
  
  if (!wifiManager.autoConnect("Abhinav's datalogger")) {
    Serial.println("failed to connect and hit timeout");
    delay(3000);
    //reset and try again, or maybe put it to deep sleep
    ESP.reset();
    delay(5000);
  }
  //if you get here you have connected to the WiFi
  Serial.println("connected...yeey :)");
  //delay(3000);
   server.begin();
   Serial.println("Server started");
  // Print the IP address
  Serial.println(WiFi.localIP());

}

void loop()
{
  // This sketch displays information every time a new sentence is correctly encoded
  if(ss.available() > 0)
    if (gps.encode(ss.read())) {
      int buttonpress = digitalRead(button);
      
      if (buttonpress == 0){
        }
    
      else
        displayInfo();

      
   if (millis() > 5000 && gps.charsProcessed() < 10)
   {
    Serial.println(F("No GPS detected: check wiring."));
    while (true);
   }
}

void displayInfo()
{
  Serial.print(F("Location: "));
  if (gps.location.isValid())
  {
    Serial.print(gps.location.lat(), 6);
    Serial.print(F(","));
    Serial.print(gps.location.lng(), 6);
    latitude = gps.location.lat();
    lat_str = String(latitude , 6);
    longitude = gps.location.lng();
    lng_str = String(longitude , 6);
  }
  else
  {
    Serial.print(F("INVALID"));
  }

  Serial.print(F("  Date/Time: "));
  if (gps.date.isValid())
  {
    Serial.print(gps.date.month());
    Serial.print(F("/"));
    Serial.print(gps.date.day());
    Serial.print(F("/"));
    Serial.print(gps.date.year());
    date_str = "";
        date = gps.date.day();
        month = gps.date.month();
        year = gps.date.year();
        if (date < 10)
          date_str = '0';
        date_str += String(date);
        date_str += " / ";
        if (month < 10)
          date_str += '0';
        date_str += String(month);
        date_str += " / ";


        if (year < 10)
          date_str += '0';
        date_str += String(year);
        Serial.println(date_str);
  }
  else
  {
    Serial.print(F("INVALID"));
  }

  Serial.print(F(" "));
  if (gps.time.isValid())
  {
    if (gps.time.hour() < 10) Serial.print(F("0"));
    Serial.print(gps.time.hour());
    Serial.print(F(":"));
    
    if (gps.time.minute() < 10) Serial.print(F("0"));
    Serial.print(gps.time.minute());
    Serial.print(F(":"));
    if (gps.time.second() < 10) Serial.print(F("0"));
    Serial.print(gps.time.second());
    Serial.print(F("."));
    if (gps.time.centisecond() < 10) Serial.print(F("0"));
    Serial.print(gps.time.centisecond());

    tim = String(gps.time.hour()) + ":" + String(gps.time.minute()) + ":" + String(gps.time.second()) + "." + String(gps.time.centisecond());
  }
  else
  {
    Serial.print(F("INVALID"));
  }

  Serial.println();
  senddata();
}

void senddata() {
  client.setInsecure();
  Serial.print("connecting to ");
  Serial.println(host);
  if (!client.connect(host, httpsPort)) {
    Serial.println("connection failed");
    return;
  }
  if (client.verify(fingerprint, host)) {
    Serial.println("certificate matches");
  } else {
    Serial.println("certificate doesn't match");
  }
  String url = "/macros/s/" + GAS_ID + "/exec?device=" + device + "&lati=" + String(gps.location.lat(), 6) + "&longi=" + String(gps.location.lng(), 6) + "&time=" + tim;
  Serial.print("requesting URL: ");
  Serial.println(url);
  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
               "Host: " + host + "\r\n" +
               "User-Agent: BuildFailureDetectorESP8266\r\n" +
               "Connection: close\r\n\r\n");
  Serial.println("request sent");
  while (client.connected()) {
    
    String line = client.readStringUntil('\n');
    Serial.println("line:"+line);
    if (line == "\r") {
      Serial.println("headers received");
      break;
    }
  }
  String line = client.readStringUntil('\n');
  if (line.startsWith("{\"state\":\"success\"")) {
    Serial.println("esp8266/Arduino CI successfull!");
  } else {
    Serial.println("esp8266/Arduino CI has failed");
  }
  Serial.println("reply was:");
  Serial.println("==========");
  Serial.println(line);
  Serial.println("==========");
  Serial.println("closing connection");
}
