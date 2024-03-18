# PR2_PD_XavierHidalgo

## PRACTICA 2: INTERRUPCIONES

### Practica A interrupción por GPIO


**PROGRAMA:**

``` cpp
#include <Arduino.h>

struct Button{
  const uint8_t PIN;
  uint32_t numberKeyPresses;
  bool pressed;
};

Button button1 = {23,0,false};

void IRAM_ATTR isr() {
  button1.numberKeyPresses += 1;
  button1.pressed = true;
}

static uint32_t lastMillis = 0;

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println(button1.PIN);
  pinMode(button1.PIN, INPUT_PULLUP);
  attachInterrupt(button1.PIN, isr, FALLING);
}

void loop() {
  if (button1.pressed) {
  Serial.printf("Button 1 has been pressed");
  button1.pressed = false;
  }
  
  //Detach Interrupt after 1 Minute
  if (millis() - lastMillis > 60000) {
  lastMillis = millis();
  detachInterrupt(button1.PIN);
  Serial.println("Interrupt Detached!");
  }
}
```

**INFORME:**

El circuito a implementar en esta práctica consta de simplemente dos cables, uno conectado al pin 23 (como podria ser cualquier otro pin salida) del y otro al pin GND (tierra) del MP.
Al hacer contacto con los dos cables permitiremos que fluya corriente por ellos hecho que el MP entenderá como una interrupción. La función que nos permite que el MP entienda esta acción como una interrupción es:
``` cpp
attachInterrupt(GPIOPin, ISR, Mode);
```
Esta función forma parte de la libreria Arduino.h. Las primeras 2 variables a definir cuando llamamos a la función simplemente definen a un pin como "pin de interrupción" (GPIOPin) y la función a llamar cuando suceda una interrupción (ISR).
La tercera variable es interesante. Define a nivel eléctrico que definimos como interrupción, por ejemplo, el paso de corriente por nuestro "pin de interrupción", el NO paso de corriente por nuestro "pin de interrupción", el cambio de corriente a no corriente, etc. Concretando y, como dice la práctica, podemos definir estas funciones: 

* LOW Los disparadores interrumpen cuando el pin está LOW
* HIGH Los disparadores interrumpen cuando el pin es HIGH
* CHANGE Los disparadores interrumpen cuando el pin cambia de valor, de HIGH a LOW o LOW a HIGH
* FALLING Los disparadores interrumpen cuando el pin va de HIGH a LOW
* RISING Los disparadores interrumpen cuando el pin va de LOW a HIGH

Como podeis ver, en nuestro programa hemos usado la función FALLING, es decir, el paso de "unir a desunir" los dos cables.

Bien, en esta práctica, la acción que se producirá cuando se genere una interrupción (cunado suceda lo explicado en la linea anterior), será simplemente enviar un mensaje por el puerto série de este tipo:
``` cpp
Serial.printf("Button 1 has been pressed");
```
Además, se sumará 1 al valor que haya en la variable **numberKeyPresses** de nuestra tupla **Button**. Mirando el programa este contador no se usa en ningún momento, pero hemos pensado que podriamos enviarlo por el puerto série junto al mensaje anterior de la siguente forma:
``` cpp
Serial.printf("Button 1 has been pressed "button1.numberKeyPresses" times");
```
Para darle un uso almenos.

Finalmente, dentro del loop, encontramos un if que desactivará la interrupción pasado un minuto, entendemos que a parte de funcionar como un control de errores, es una forma de usar la función detachInterrupt.
