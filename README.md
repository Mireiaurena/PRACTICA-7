# PRACTICA-7
# Práctica 7.1: Reproducción de Audio con I2S

## Propósito:
- Familiarizarse con el funcionamiento del bus I2S mediante la reproducción de un archivo de audio.
- Implementar una prueba para reproducir sonido a través de un altavoz utilizando un archivo almacenado en la memoria interna.

## Elementos Necesarios:
- ESP32-S3
- Altavoz compatible con I2S
- Amplificador compatible con I2S

## Procedimiento:

### Código:
```cpp
#include <Arduino.h>
#include "AudioGeneratorAAC.h"
#include "AudioOutputI2S.h"
#include "AudioFileSourcePROGMEM.h"
#include "sampleaac.h"

AudioFileSourcePROGMEM *in;
AudioGeneratorAAC *aac;
AudioOutputI2S *out;

void setup()
{
    Serial.begin(115200);
    audioLogger = &Serial;
    in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac));
    aac = new AudioGeneratorAAC();
    out = new AudioOutputI2S();
    aac->begin(in, out);
}

void loop()
{
    if (aac->isRunning()) {
        aac->loop();
    } else {
        Serial.printf("AAC done\n");
        delay(1000);
    }
}
```

### Explicación:
El programa carga y reproduce un archivo de audio en formato AAC almacenado en la memoria flash del ESP32-S3 mediante una salida de audio I2S.

1. **Inicialización:** En la función `setup()`, se configura la comunicación serial a 115200 baudios para propósitos de depuración. Luego, se define `audioLogger` para redirigir mensajes de depuración a la consola.
2. **Carga del archivo:** Se crea una instancia de `AudioFileSourcePROGMEM`, donde se indica el archivo de audio (`sampleaac`) y su tamaño.
3. **Configuración del audio:** Se instancian el generador de audio AAC (`AudioGeneratorAAC`) y la salida I2S (`AudioOutputI2S`). Posteriormente, se inicia la reproducción con `aac->begin(in, out)`, que enlaza la fuente de audio con la salida.
4. **Ejecución en bucle:** En la función `loop()`, si el generador de audio sigue activo, se llama a `aac->loop()`. Una vez finalizada la reproducción, se imprime "AAC done" en la consola serial y se espera un segundo antes de repetir el ciclo.

## Conclusión:
Este ejercicio demuestra la capacidad del ESP32-S3 para reproducir archivos de audio AAC almacenados en memoria interna a través de una interfaz I2S, proporcionando una base sólida para futuras implementaciones de audio en proyectos embebidos.

