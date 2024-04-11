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

El circuito a implementar en esta práctica consta de simplemente dos cables, uno conectado al pin 23 (como podria ser cualquier otro pin GPIO (general propuse input/output)) y otro al pin GND (tierra) del MP.
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

Como podeis ver, en nuestro programa hemos usado la función FALLING, es decir, el paso "de unir a desunir" los dos cables.

En nuestro programa utilizamos una tupla (struct) llamada Button. Esta tupla consta de 3 variables; una que define el "pin de interrupcion" (GPIO), otra variable contadora (más adelante hablamos de ella) y otra booleana (la explicamos en la siguiente linea). Seguidamente definimos una variable tipo Button llamada button1.   

Bien, cuando se genere una interrupción (cunado suceda la unión-desunión entre los cables), se llamará a la función ISR como ya dijimos. Esta simplemente cambiara el valor de la variable booleana de button1 a true, es decir, es una variable que **define si ha habido interrupción**. Además sumara +1 a la variable contadora de button1.

**Es importante** diferenciar entre lo que sucede cuando se produce una interrupción, es decir, lo que hay dentro de la función ISR, y las consecuencias que pueden conllevar las acciones tomadas en ISR. En nuestro caso, si nos fijamos en nuestro void loop(), estamos preguntando constantemente si la variable booleana de button1 es true, hecho que solo sucede si ha habido una interrupcion y se ejecuta la función ISR. Vemos entonces que lo que haya dentro del if del loop() se ejecurtara cuando haya una interrupción, pero es importante diferenciar entre lo que nosotros ejecutamos tras una interrupción, que seria la función ISR, y lo que puede conllevar las acciones de dicha función, que seria la ejecución del if del loop(). 

En este caso, cuendo se cumpla dicho if simplemente enviaremos un mensaje por el puerto série de este tipo:
``` cpp
Serial.printf("Button 1 has been pressed");
```
Y cambiaremos el valor de la variable booleana de button1 (para que no se quede siempre en true claro).

Si nos fijamos, hemos comentado solo una consecuencia de las acciones tomadas en la función ISR, pero esta define dos acciones, que pasa con la de sumar +1 a la variable contadora de button1? Bien, mirando el programa este contador no se usa en ningún momento, pero hemos pensado que podriamos enviarlo por el puerto série junto al mensaje anterior de la siguente forma:
``` cpp
Serial.printf("Button 1 has been pressed "button1.numberKeyPresses" times");
```
Para darle un uso, que menos.

Finalmente, dentro del loop, encontramos un if que desactivará la interrupción pasado un minuto, entendemos que a parte de funcionar como un control de errores, es una forma de usar la función detachInterrupt.

### Practica B interrupción por timer


**PROGRAMA:**

``` cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;

hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR onTimer() {
 portENTER_CRITICAL_ISR(&timerMux);
 interruptCounter++;
 portEXIT_CRITICAL_ISR(&timerMux);
}
void setup() {

 Serial.begin(115200);
 
 timer = timerBegin(0, 80, true);
 timerAttachInterrupt(timer, &onTimer, true);
 timerAlarmWrite(timer, 1000000, true);
 timerAlarmEnable(timer);
}
void loop() {
 if (interruptCounter > 0) {
  portENTER_CRITICAL(&timerMux);
  interruptCounter--;
  portEXIT_CRITICAL(&timerMux);
  
  totalInterruptCounter++;
  
  Serial.print("An interrupt as occurred. Total number: ");
  Serial.println(totalInterruptCounter);
 }
}
```

En este programa seguimos usando interrupciones pero hemos cambiado el tipo de interrupción. Ahora usamos interrupciones de tipo temporizador o "timer" en vez de GPIO o de "pin de interrupción".


Bien, el programa es muy sencillo, simplemente cada microsegundo (timepo modificable) se produce una interrupción automática. La acción que tomamos cada vez que se produce una interrupción, es decir, cada microsegundo, es sumar +1 a una variable llamada interruptCounter.

Ojo! Esta es la acción que se ejecuta automáticamente cuando se produce una interrupción. Si recordais el ejercicio anterior, diferenciavamos entre las acciones que ejecutábamos cuando se producía una interrupción, lo que habia dentro de **ISR**, y las consecuencias que estas producian, que las podíamos ver dentro del **loop()** del programa.

En este ejercicio pasa lo mismo, lo único es que la función ISR se ha pasado a llamar **onTimer**. Cuando se produce una interrupción llamamos a onTimer y esta suma +1 a la varibale nombrada anteriormente interruptCounter. Fijemonos en el void loop(), hay un if que se ejecutara cuando interruptCounter sea mayor que 0 y esto solo pasará cuando se produzca una interrupción (básicamente lo mismo que el ejercicio anterior). 

Que se ejecuta dentro de este if? Pues sumaremos +1 a otra variable llamada totalInterruptCounter y restaremos +1 a la variable interruptCounter para que se pueda volver a ejecutar este if (como hacíamos en el ejercicio anterior pero en vez de con una variable booleana con una integer). Tambien enviaremos un mensaje por el puerto série que nos irá informando de cuantas interrupciones han ocurrido, esto lo hará gracias a totalInterruptCounter:

``` cpp
Serial.print("An interrupt as occurred. Total number: ");
Serial.println(totalInterruptCounter);
```
Podemos modificar el tiempo en el que ocurre una interrupción en el 100000 de:
``` cpp
 timerAlarmWrite(timer, 1000000, true);
```
Como veis en este programa seguimos utilizando funciones propias de Arduino de interrupciones, solo que ahora de tipo timer (timerAttachInterrupt, timerAlarmWrite, timerAlarmEnable, etc).
