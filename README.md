# blown-up-mind
//Load cell_Saline_bottle_iotbased measuring _web data push _monitoring
//Includes / Libraries
#include "HX711.h"
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
//Pins
#define DOUT  D3
#define CLK  D2
//Cloud Constants
#define URL "http://api.thingspeak.com/update?"
#define SECRET_WRITE_APIKEY "5IECQOBVV3NW01ZW"   // replace XYZ with your channel write API Key
//Network Constants
const char *ssid =  "cisco";
const char *pass =  "6933220l";
//Sensor Objects
HX711 scale;
float calibration_factor = 419;
//Network Objects
WiFiClient client;



//Init OS
void setup() {
  Serial.begin(115200);
  Serial.println("HX711 calibration sketch");
  Serial.println("Remove all weight from scale");
  Serial.println("After readings begin, place known weight on scale");
  Serial.println("Press + or a to increase calibration factor");
  Serial.println("Press - or z to decrease calibration factor");

  scale.begin(DOUT, CLK);
  scale.set_scale();
  scale.tare(); //Reset the scale to 0

  long zero_factor = scale.read_average();
  Serial.print("Zero factor: ");
  Serial.println(zero_factor);

  Serial.println("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, pass);

  while (WiFi.status() != WL_CONNECTED) //thingseak ka client void setup pe nh diya gya hh
  {
    delay(500);   //islye nhi ho rha hh shyd copy it then ck ok
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
}

void loop() {
  long scaleVal = getScale();
  long values[] = {scaleVal,0/*Other fields go here...*/};
  sendToCloud(values);
}

long getScale() {
  getKeyboard();
  scale.set_scale(calibration_factor);
  Serial.print("Reading: ");
  long res = scale.get_units();
  Serial.print(res);
  Serial.print(" gms");
  Serial.print(" calibration_factor: ");
  Serial.print(calibration_factor);
  Serial.println();
 
  return res;
}

void getKeyboard() {
  if (Serial.available())
  {
    char temp = Serial.read();
    if (temp == '+' || temp == 'a')
      calibration_factor += 10;
    else if (temp == '-' || temp == 'z')
      calibration_factor -= 10;
  }
}

void sendToCloud(long fields[]) {
  HTTPClient http;
  http.begin(String(URL)+"api_key="+SECRET_WRITE_APIKEY+"&field1="+String(fields[0])); //HTTP
  int httpCode = http.GET();
  // httpCode will be negative on error
  if (httpCode > 0) {
    if (httpCode == HTTP_CODE_OK) {
      String payload = http.getString();
      Serial.println("Success: "+payload);
    }
  } else {
    Serial.println("Failed");
  }
  http.end();
}
