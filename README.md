# capo

#include <Arduino.h>
#include <Wire.h>
#include <SPI.h>
#include <Adafruit_Sensor.h>
#include "Adafruit_TSL2591.h"

Adafruit_TSL2591 tsl = Adafruit_TSL2591(2591); // pass in a number for the sensor identifier (for your use later)
 
 
/**************************************************************************/
/*
    Configures the gain and integration time for the TSL2591
*/
/**************************************************************************/
void configureSensor(void)
{
  // You can change the gain on the fly, to adapt to brighter/dimmer light situations
  //tsl.setGain(TSL2591_GAIN_LOW);    // 1x gain (bright light)
  tsl.setGain(TSL2591_GAIN_MED);      // 25x gain
  //tsl.setGain(TSL2591_GAIN_HIGH);   // 428x gain
  tsl.setTiming(TSL2591_INTEGRATIONTIME_300MS);
 
 
  /* Display the gain and integration time for reference sake */  
  Serial.println(F("------------------------------------"));
  Serial.print  (F("Gain:         "));
  tsl2591Gain_t gain = tsl.getGain();
  switch(gain)
  {
    case TSL2591_GAIN_LOW:
      Serial.println(F("1x (Low)"));
      break;
    case TSL2591_GAIN_MED:
      Serial.println(F("25x (Medium)"));
      break;
    case TSL2591_GAIN_HIGH:
      Serial.println(F("428x (High)"));
      break;
    case TSL2591_GAIN_MAX:
      Serial.println(F("9876x (Max)"));
      break;
  }
  Serial.print  (F("Timing:       "));
  Serial.print((tsl.getTiming() + 1) * 100, DEC); 
  Serial.println(F(" ms"));
  Serial.println(F("------------------------------------"));
  Serial.println(F(""));
}

int conversion(int array[], int len) {
    int output = 0;
    int power = 1;

    for (int i = 0; i < len; i++)
    {
        output += array[(len - 1) - i] * power;
        // output goes 1*2^0 + 0*2^1 + 0*2^2 + ...
        power *= 2;
    }

    return output;
}

void simpleRead(void)
{
  // Simple data read example. Just read the infrared, fullspecrtrum diode 
  // or 'visible' (difference between the two) channels.
  // This can take 100-600 milliseconds! Uncomment whichever of the following you want to read
  uint16_t x = tsl.getLuminosity(TSL2591_VISIBLE);
  //uint16_t x = tsl.getLuminosity(TSL2591_FULLSPECTRUM);
  //uint16_t x = tsl.getLuminosity(TSL2591_INFRARED);
 
  Serial.print(F("[ ")); Serial.print(millis()); Serial.print(F(" ms ] "));
  Serial.print(F("Luminosity: "));
  Serial.println(x, DEC);
}

uint16_t luzAmbiente = 0;
uint16_t luzAmbienteTotal=0;
uint16_t luzAmbienteTotalSinLed = 0;

/*void calcularLuzAmbiente(){
  Serial.println("Calculando luz ambiente 1");
  luzAmbiente = tsl.getLuminosity(TSL2591_VISIBLE);
  luzAmbienteTotalSinLed+= luzAmbiente;
  delay(1000);
  Serial.println("Calculando luz ambiente 2");
  luzAmbiente = tsl.getLuminosity(TSL2591_VISIBLE);
  luzAmbienteTotalSinLed += luzAmbiente;
  delay(1000);
  Serial.println("Calculando luz ambiente 3");
  luzAmbiente = tsl.getLuminosity(TSL2591_VISIBLE);
  luzAmbienteTotalSinLed += luzAmbiente;
  //La luz vale aprox. 300?
  luzAmbienteTotalSinLed/=3;
  Serial.print("Sin led: ");
  Serial.println(luzAmbienteTotalSinLed);
  digitalWrite(ledPIN , HIGH);
  Serial.println("Calculando luz ambiente 4");
  luzAmbiente = tsl.getLuminosity(TSL2591_VISIBLE);
  luzAmbienteTotal += luzAmbiente;
  delay(1000);
  Serial.println("Calculando luz ambiente 5");
  luzAmbiente = tsl.getLuminosity(TSL2591_VISIBLE);
  luzAmbienteTotal += luzAmbiente;
  delay(1000);
  Serial.println("Calculando luz ambiente 6");
  luzAmbiente = tsl.getLuminosity(TSL2591_VISIBLE);
  luzAmbienteTotal += luzAmbiente;
  //La luz vale aprox. 300?
  luzAmbienteTotal/=3;
  Serial.print("Con led: ");
  Serial.println(luzAmbienteTotal);
  Serial.print("El valor aproximado del led: ");
  Serial.println(luzAmbienteTotal-luzAmbienteTotalSinLed);
  luzAmbienteTotal = luzAmbienteTotal - ((luzAmbienteTotal-luzAmbienteTotalSinLed)/2);
}*/

bool sensorLectura(){
    uint16_t x = tsl.getLuminosity(TSL2591_VISIBLE);
    if(x<7000) {
      return false;
    }
    else {
      return true;
    }
}

/*void convertirMensajeAPaquete(){
  String msg = "hola";
  String myText = "a";

  for(int i=0; i<myText.length(); i++){

    char myChar = myText.charAt(i);
    
      for(int i=7; i>=0; i--){
        byte bytes = bitRead(myChar,i);
        if (bytes==1)   digitalWrite(ledPIN , HIGH);
        else   digitalWrite(ledPIN , LOW);
        Serial.print(sensorLectura());
        Serial.println("<--- el sensor");
        Serial.println(bytes);
        delay(200);
      }

      Serial.println("");
  }
  Serial.println("Fin");
  digitalWrite(ledPIN , LOW);
  delay(3000);
} */

bool comprobarProtocolo(){
  bool valorProtocolo = false;
  for(int i = 0; i<7; i++){
    if (sensorLectura()==valorProtocolo) valorProtocolo = !valorProtocolo;
    else return false;
  }
  return true;
}

void funcionLeer(){
  //15.000 con linterna
  int lista[8];
  for (int i = 0; i<8; i++){
    Serial.println("Ubicar la luz");
    Serial.println("Leyendo...");
    uint16_t x = tsl.getLuminosity(TSL2591_VISIBLE);
    if(x>4600) {
      Serial.println("Leo 1");
      lista[i] = 1;
    }
    else {
      Serial.println("Leo 0");
      lista[i] = 0;
    }
  }

  for(int i=0;i<8;i++){
    Serial.print("[");
    Serial.print(lista[i]);
    Serial.print("]");
  }
  Serial.println("--> Array");

  Serial.println(conversion(lista, 8));
}

void setup() {
  Serial.begin(9600);
  Serial.println(F("Starting Adafruit TSL2591 Test!"));

   if (tsl.begin()) 
  {
    Serial.println(F("Found a TSL2591 sensor"));
  } 
  else 
  {
    Serial.println(F("No sensor found ... check your wiring?"));
    while (1);
  } 

  configureSensor();

}

void loop() {
   //funcionLeer();
   //simpleRead();
   if (sensorLectura()) comprobarProtocolo();
   delay(5);
}


