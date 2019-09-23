
## Install tensorFlow lite from source  

Posiblemente nos quedemos defraudados  luego de instalar la librería de tensorFlow lite en el entorno de arduino , pues no podemos modificar el código fácilmente. Pero lo podemos  solucionarlo de dos modos: el fácil y el más fácil.  

La solución fácil la podemos encontrar  tácitamente en la información del [repositorio](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/lite/experimental/micro/examples/hello_world).  

> “If you need to build the library from source (for example, if you're making modifications to the code), run this command to generate a zip file containing the required source files:”  

```bash
make -f tensorflow/lite/experimental/micro/tools/make/Makefile TARGET=arduino TAGS="" generate_hello_world_arduino_library_zip
```  

Luego nos indica dónde podemos encontrar el archivo comprimido generado:  

```bash
tensorflow/lite/experimental/micro/tools/make/gen/arduino_x86_64/prj/hello_world/hello_world.zip
```   

Lamentablemente omite que es realmente necesario para poder generar __NUESTRO__ propio código en el Ide de arduino. Afortunadamente  no debemos modificar el código actual de la librería almacenada en GitHub (y espero que se mantenga así) , solo debemos generar el archivo zip.    

Para poder generar la librería debemos bajar en repositorio con la indicación ```--depth 1 ``` , como se indica:  

```bash
git clone --depth 1 https://github.com/tensorflow/tensorflow.git
```  

Para desarrollar este proceso, aquí  tenemos varias posibilidades:  
1.	Hacer lo el procedimiento en un Linux.
2.	Hacerlo en una maquina virtual(Win)
3.	Hacerlo en Docker (Win, Linux, Mac).
4.	En una instancia de AWS .
5.	Hacerlo en Colab.  

No es posible hacerlo en Windows por que requiere  __Make 3.82__, y la última versión disponible para [Window es la 3.81](http://gnuwin32.sourceforge.net/packages/make.htm).  

O el modo más fácil es bajar la versión compilada que tengo almacenada en este [repositorio](https://github.com/sandroormeno/share_aws/blob/master/hello_world.zip?raw=true).  

Una vez que tengamos la librería podemos abrir el ejemplo, _et voilà_, ya podemos ver el código dentro del IDE de arduino. Aquí una breve descripción de los archivos:  

####  Hello_world.ino  

Tiene todo el código necesario para correr la red neuronal  desarrollada  con Tensorflow , pero almacenada en un formato ligero denominado LITE. Es lo único que requerimos para correr nuestro entrenamiento más los archivos del modelo.  

####  Sine_model_data.cpp y sine_model_data.h  

En ambos archivos se almacena la información del modelo. El arreglo hexadecimal  de  datos del modelo. Este archivo guarda las bias y weights de las capas de neuronas. Además de la estructura de modelo construido en Keras.   

####  Arduino_constants.cpp y constants.h  

En estos archivos tenemos algunas constantes para poder generar los datos de entrada. Debemos recordar que los datos fueron generados con una pequeña función con python en jupyter notebook . En este caso debemos generar el valor “X” para que nuestro modelo nos devuelva un valor “Y” que generará la curva seno, que es el objetivo del ejemplo.  

####  Arduino_output_handler.cpp y output_handler.h  

Estos  archivos  definen la función de salida de ejemplo: los valores generados por la red neuronal se vinculan al LED13  e imprimen  este valor en el monitor serie.  Que fue lo que modificamos en la experiencia  del tutorial de __youtube__, que fue básicamente incluir la salida serial e imprimir el valor de brightness:  

```c++

#include "Arduino.h"
#include "constants.h"

// The pin of the Arduino's built-in LED
int led = LED_BUILTIN;

// Track whether the function has run at least once
bool initialized = false;

// Animates a dot across the screen to represent the current x and y values
void HandleOutput(tflite::ErrorReporter* error_reporter, float x_value,
                  float y_value) {
  // Do this only once
  if (!initialized) {
    // Set the LED pin to output
    Serial.begin(9600); // <---- primer cambio
    pinMode(led, OUTPUT);
    initialized = true;
  }

  // Calculate the brightness of the LED such that y=-1 is fully off
  // and y=1 is fully on. The LED's brightness can range from 0-255.
  int brightness = (int)(127.5f * (y_value + 1));

  // Set the brightness of the LED. If the specified pin does not support PWM,
  // this will result in the LED being on when y > 127, off otherwise.
  analogWrite(led, brightness);

  // Log the current brightness value for display in the Arduino plotter
  error_reporter->Report("%d\n", brightness);
  Serial.println(brightness); // <---- segundo cambio
}

```






    
    
