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
#include "AudioGeneratorAAC.h" 
#include "AudioOutputI2S.h" 
#include "AudioFileSourcePROGMEM.h" 
#include "sampleaac.h" 
AudioFileSourcePROGMEM *in; 
AudioGeneratorAAC *aac; 
AudioOutputI2S *out; 
void setup(){ 
Serial.begin(115200); 
in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac)); 
aac = new AudioGeneratorAAC(); 
out = new AudioOutputI2S(); 
out -> SetGain(0.125); 
out -> SetPinout(40,39,38); 
aac->begin(in, out); 
} 
void loop(){ 
if (aac->isRunning()) { 
aac->loop(); 
} else { 
aac -> stop(); 
Serial.printf("Sound Generator\n"); 
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

// Pines adaptados para ESP32-S3
#define SD_CS         10  
#define SPI_MOSI      11  
#define SPI_MISO      13  
#define SPI_SCK       12  
#define I2S_DOUT      6   
#define I2S_BCLK      5   
#define I2S_LRC       4   

Audio audio; 

void setup() {
    Serial.begin(115200);

    // Inicializar la SD
    pinMode(SD_CS, OUTPUT);
    digitalWrite(SD_CS, HIGH);
    if (!SD.begin(SD_CS)) {
        Serial.println("Error al inicializar la tarjeta SD");
        return;
    }
    Serial.println("Tarjeta SD lista.");

    // Crear un archivo WAV en la SD
    createWavFile("sonido.wav");

    // Configurar el audio
    audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT);
    audio.setVolume(10); // 0...21
    audio.connecttoFS(SD, "sonido.wav");
}

void loop() {
    audio.loop();
}

// Crear un archivo WAV simple en la SD
void createWavFile(const char *filename) {
    File file = SD.open(filename, FILE_WRITE);
    if (!file) {
        Serial.println("Error al crear el archivo WAV");
        return;
    }

    // Escribir cabecera WAV (PCM, 16 bits, 44.1 kHz, mono)
    uint8_t wavHeader[44] = {
        'R','I','F','F', 36,0,0,0, 'W','A','V','E', 'f','m','t',' ',
        16,0,0,0, 1,0, 1,0, 0x44,0xAC,0x00,0x00, 0x88,0x58,0x01,0x00,
        2,0,16,0, 'd','a','t','a', 0,0,0,0
    };
    file.write(wavHeader, 44);

    // Escribir datos de audio (tono simple)
    for (int i = 0; i < 44100; i++) { 
        int16_t sample = (int16_t)(sin(2.0 * PI * 440.0 * i / 44100) * 32767);
        file.write((uint8_t*)&sample, 2);
    }

    file.seek(40); // Ir a la posición del tamaño del archivo
    uint32_t dataSize = 44100 * 2; // 1 segundo de audio mono 16-bit
    file.write((uint8_t*)&dataSize, 4);

    file.close();
    Serial.println("Archivo WAV creado: sonido.wav");
}
```

### Descripción:
El código permite la conexión del ESP8266/ESP32 a una red WiFi y la configuración de un servidor web para controlar la reproducción de radio en línea. Se utilizan bibliotecas para manejar el streaming de audio en formatos MP3 y AAC, empleando salida I2S para conectar un altavoz.

En la función `setup()`, se configura la conexión WiFi y se inicia el servidor web. También se inicializa la reproducción de audio utilizando `AudioOutputI2S`. 

En el `loop()`, se monitorea la llegada de nuevas solicitudes HTTP para actualizar la URL de la radio o controlar el volumen. Además, se mantiene en ejecución el procesamiento del audio en tiempo real.

## Conclusión:
Este proyecto demuestra cómo utilizar ESP8266/ESP32 para reproducir audio en streaming desde una fuente en línea, controlándolo a través de una interfaz web. Es una aplicación útil para proyectos de radio en red de bajo consumo y fácil implementación.

