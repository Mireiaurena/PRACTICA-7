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

# Practica 7.2: Radio Web con ESP8266/ESP32

## Objetivo:
Este proyecto implementa una radio web utilizando ESP8266 o ESP32. Se conecta a una red WiFi y permite la reproducción de archivos de audio en streaming en formatos MP3 y AAC mediante una interfaz web.

## Materiales Requeridos:
- ESP8266 o ESP32
- Conexión WiFi
- Altavoz con salida I2S

## Procedimiento:

### Código:
```cpp
#include "Audio.h"
#include "SD.h" 
#include "FS.h" 

// Digital I/O used 
#define SD_CS 5          
#define SPI_MOSI 23      
#define SPI_MISO 19      
#define SPI_SCK 18       
#define I2S_DOUT 25      
#define I2S_BCLK 27      
#define I2S_LRC 26       

Audio audio; 

void setup(){ 
    pinMode(SD_CS, OUTPUT); 
    digitalWrite(SD_CS, HIGH); 
    SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI); 
    Serial.begin(115200); 
    SD.begin(SD_CS); 
    audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT); 
    audio.setVolume(10); // 0...21  
    audio.connecttoFS(SD, "Ensoniq-ZR-76-01-Dope-77.wav"); 
} 

void loop(){ 
    audio.loop(); 
} 

// optional 
void audio_info(const char *info){ 
    Serial.print("info        "); Serial.println(info); 
} 
void audio_id3data(const char *info){  //id3 metadata 
    Serial.print("id3data     ");Serial.println(info); 
} 
void audio_eof_mp3(const char *info){  //end of file 
    Serial.print("eof_mp3     ");Serial.println(info); 
} 
void audio_showstation(const char *info){ 
    Serial.print("station     ");Serial.println(info); 
} 
void audio_showstreaminfo(const char *info){ 
    Serial.print("streaminfo  ");Serial.println(info); 
} 
void audio_showstreamtitle(const char *info){ 
    Serial.print("streamtitle ");Serial.println(info); 
} 
void audio_bitrate(const char *info){ 
    Serial.print("bitrate     ");Serial.println(info); 
} 
void audio_commercial(const char *info){  //duration in sec 
    Serial.print("commercial  ");Serial.println(info); 
} 
void audio_icyurl(const char *info){  //homepage 
    Serial.print("icyurl      ");Serial.println(info); 
} 
void audio_lasthost(const char *info){  //stream URL played 
    Serial.print("lasthost    ");Serial.println(info); 
} 
void audio_eof_speech(const char *info){ 
    Serial.print("eof_speech  ");Serial.println(info); 
} 
```

### Descripción:
El código permite la conexión del ESP8266/ESP32 a una red WiFi y la configuración de un servidor web para controlar la reproducción de radio en línea. Se utilizan bibliotecas para manejar el streaming de audio en formatos MP3 y AAC, empleando salida I2S para conectar un altavoz.

En la función `setup()`, se configura la conexión WiFi y se inicia el servidor web. También se inicializa la reproducción de audio utilizando `AudioOutputI2S`. 

En el `loop()`, se monitorea la llegada de nuevas solicitudes HTTP para actualizar la URL de la radio o controlar el volumen. Además, se mantiene en ejecución el procesamiento del audio en tiempo real.

## Conclusión:
Este proyecto demuestra cómo utilizar ESP8266/ESP32 para reproducir audio en streaming desde una fuente en línea, controlándolo a través de una interfaz web. Es una aplicación útil para proyectos de radio en red de bajo consumo y fácil implementación.

