#include <Wire.h>
#include "rgb_lcd.h"
#include "IotHttpClient.h"
#include "IotUtils.h"

#include <LWiFi.h>
#include <string>
#include <rgb_lcd.h>
// ********************************************
// ****** START user defined definitions ******
// ********************************************
#define WIFI_SSID                     "DeGea"  //"TP-IOT"
#define WIFI_PASSWORD                 "broklyncat95"      //"iot@tp4me"

#define pinred 2
#define pinblue 4
#define pingreen 3
#define Touch 8
#define fsrPin A0
int val = 0;     // val will be used to store the state
// of the input pin
int old_val = 0; // this variable stores the previous
// value of "val"
int state = 0;   // 0 = pinred off and 1 = pinred on

int fsrReading;

// ******************************************
// ****** END user defined definitions ******
// ******************************************

//#define DELAY 60000

rgb_lcd lcd;
const int colorR = 0;
const int colorG = 255;
const int colorB = 0;

// ******************************************
// ****** setup()****************************
// ******************************************
void setup()
{
  Serial.begin(9600);
  pinMode(pinred, OUTPUT);   // tell Arduino LED is an output
  pinMode(pingreen, OUTPUT);
  pinMode(pinblue, OUTPUT);
  pinMode(Touch, INPUT); // and BUTTON is an input
  pinMode(fsrPin, INPUT);
  lcd.begin(16, 2);
  lcd.setRGB(colorR, colorG, colorB);
  LWiFi.begin();
  delay(6000);


}

// ******************************************
// ****** loop() ****************************
// ******************************************
void loop()
{

  Serial.print("\nSearching for Wifi AP...\n");

  if ( LWiFi.connectWPA(WIFI_SSID, WIFI_PASSWORD) != 1 )
  {
    Serial.println("Failed to Connect to Wifi.");
  }
  else
  {
    Serial.println("Connected to WiFi");
    // Generate some random data to send to Azure cloud.
    srand(vm_ust_get_current_time());

    // This method calls an internal function that detects the state of the button
    int table_id = 1 + (rand() % 50);
    int astatus = old_val;
    int old_val = 0;

    // Construct a JSON data string using the random data.

    std::string json_iot_data;
    json_iot_data += "{ \"TableId\":" + IotUtils::IntToString(table_id);   
    std::string taken = "Taken";
    std::string clean = "Cleanning";
    std::string aval = "available";
    // check if there was a transition
    val = digitalRead(Touch); // read input value and store it
    if ((val == HIGH) && (old_val == LOW)) {
      state = 1 - state;

    }

    old_val = val;  // val is now old, let's store it

    if (state == 1) {
      fsrReading = analogRead(fsrPin);
      if (fsrReading > 10) {
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Taken");
        Serial.println("Taken");
        digitalWrite(pinred, HIGH);
        digitalWrite(pingreen, LOW);
        digitalWrite(pinblue, LOW);
        lcd.setRGB(colorR, colorG, colorB);
        json_iot_data += ", \"Status\":\"" + taken + "\"";
        json_iot_data += " }";
      }
      else {
        digitalWrite(pinred, LOW);
        digitalWrite(pinblue, HIGH);
        digitalWrite(pingreen, LOW);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("cleaning");
        lcd.setRGB(255, 255, 255);
        json_iot_data += ", \"Status\":\"" + clean + "\"";
        json_iot_data += " }";

      }
    }
    else {
      lcd.setCursor(0, 0);
      lcd.print("Available");
      Serial.println("Avaialble");
      digitalWrite(pinred, LOW);
      digitalWrite(pingreen, HIGH);
      digitalWrite(pinblue, LOW);
      lcd.setRGB(0, 255, 34);
      //json_iot_data += ", \"Status\":" + aval;
      json_iot_data += ", \"Status\":\"" + aval + "\"";
      json_iot_data += " }";
    }
  

  // Construct a JSON data string using the random data.

  // Send the JSON data to the Azure cloud and get the HTTP status code.
  IotHttpClient     https_client;

  int http_code = https_client.SendAzureHttpsData(json_iot_data);

  Serial.println("Print HTTP Code:" + String(http_code));
  //lcd.setCursor(0, 1);
  //lcd.print("Code:" + String(http_code));

  //  Serial.println("Disconneting HTTP connection");
  LWiFi.disconnect();

  // Sleeps for a while...
  delay(1000);
}

}


