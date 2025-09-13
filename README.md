# Practice 3 Extra: ESP32 Web Server with 2 LEDs (English Version)

## Materials
- ESP32
- 2 LEDs
- 2 resistors (470Ω – 1kΩ)

## Introduction
In this practice, for my Digital Processors course, we learn how to use the Wi-Fi peripheral of the ESP32 to create a web server. This server allows controlling 2 LEDs: each LED has its own ON/OFF button on the web page. When the corresponding button is clicked, the LED turns on or off.

## Code Explanation (with line-by-line comments)
```cpp
#include <WiFi.h>

// SSID and Password
const char* ssid = "Mi Ada";
const char* password = "telefono4ada";

// Web server on port 80
WiFiServer server(80);

// Store HTTP request
String header;

// Output states
String output26State = "off";
String output27State = "off";

// GPIO pins for LEDs
const int output26 = 26;
const int output27 = 27;

// Timing variables
unsigned long currentTime = millis();
unsigned long previousTime = 0;
const long timeoutTime = 2000;

void setup() {
  Serial.begin(115200);

  pinMode(output26, OUTPUT);
  pinMode(output27, OUTPUT);
  digitalWrite(output26, LOW);
  digitalWrite(output27, LOW);

  // Connect to Wi-Fi
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  server.begin();
}

void loop() {
  WiFiClient client = server.available();

  if (client) {
    currentTime = millis();
    previousTime = currentTime;
    Serial.println("New Client.");
    String currentLine = "";

    while (client.connected() && currentTime - previousTime <= timeoutTime) {
      currentTime = millis();
      if (client.available()) {
        char c = client.read();
        Serial.write(c);
        header += c;
        if (c == '\n') {
          if (currentLine.length() == 0) {
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println("Connection: close");
            client.println();

            // Control LEDs
            if (header.indexOf("GET /26/on") >= 0) {
              output26State = "on";
              digitalWrite(output26, HIGH);
            } else if (header.indexOf("GET /26/off") >= 0) {
              output26State = "off";
              digitalWrite(output26, LOW);
            }

            if (header.indexOf("GET /27/on") >= 0) {
              output27State = "on";
              digitalWrite(output27, HIGH);
            } else if (header.indexOf("GET /27/off") >= 0) {
              output27State = "off";
              digitalWrite(output27, LOW);
            }

            // Web page HTML
            client.println("<!DOCTYPE html><html>");
            client.println("<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">");
            client.println("<link rel=\"icon\" href=\"data:,\">");
            client.println("<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}");
            client.println(".button { background-color: #4CAF50; border: none; color: white; padding: 16px 40px;");
            client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
            client.println(".button2 {background-color: #555555;}</style></head>");
            client.println("<body><h1>ESP32 Web Server</h1>");
            
            // Buttons for LED 26
            client.println("<p>GPIO 26 - State " + output26State + "</p>");
            if (output26State=="off") {
              client.println("<p><a href=\"/26/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/26/off\"><button class=\"button button2\">OFF</button></a></p>");
            }

            // Buttons for LED 27
            client.println("<p>GPIO 27 - State " + output27State + "</p>");
            if (output27State=="off") {
              client.println("<p><a href=\"/27/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/27/off\"><button class=\"button button2\">OFF</button></a></p>");
            }

            client.println("</body></html>");
            client.println();
            break;
          } else {
            currentLine = "";
          }
        } else if (c != '\r') {
          currentLine += c;
        }
      }
    }
    header = "";
    client.stop();
    Serial.println("Client disconnected.");
    Serial.println("");
  }
}
```


## How to Run

1. **Install PlatformIO**
   - Install [VS Code](https://code.visualstudio.com/)
   - Install the [PlatformIO extension](https://platformio.org/install/ide?install=vscode)

2. **Create a New Project**
   - Open PlatformIO in VS Code
   - Create a new project and select your board (e.g., ESP32 Dev Module)
   - Choose **Arduino framework**

3. **Add the Code**
   - Replace the contents of `src/main.cpp` with the code provided above

4. **Connect the Hardware**
   - Connect the ESP32 board to your computer via USB
   - Connect **LED1** to GPIO 26 with a resistor (470Ω–1kΩ) in series
   - Connect **LED2** to GPIO 27 with a resistor (470Ω–1kΩ) in series

5. **Build and Upload**
   - Click **Build (✓)** to compile the code
   - Click **Upload (→)** to flash the code to your ESP32

6. **Observe the Output**
   - Open a browser and navigate to the ESP32 IP shown in the Serial Monitor
   - Use the ON/OFF buttons on the web page to control LED1 and LED2

### Resources
- **Video Demonstration in Spanish:** [Watch video](assets/practica3extra.mp4)  
- **ESP32 Documentation:** [Espressif ESP32](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/index.html)  
- **ESP32 Wi-Fi Documentation:** [ESP32 Wi-Fi API](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/network/esp_wifi.html)  
- **Arduino Wi-Fi Library Reference:** [Arduino WiFi](https://www.arduino.cc/en/Reference/WiFi)  


# Práctica 3 Extra: Servidor Web ESP32 con 2 LEDs (Versión en Español)

## Materiales
- ESP32
- 2 resistencias (470Ω – 1kΩ)
- 2 LEDs


## Introducción

En esta práctica, para mi curso de Procesadores Digitales, aprendemos a usar el periférico Wi-Fi del ESP32 para crear un servidor web. Este servidor permite controlar 2 LEDs: cada LED tiene su propio botón ON/OFF en la página web. Al hacer clic en un botón, el LED correspondiente se enciende o apaga.

## Explicación del código (comentarios línea por línea):

```cpp
// libreria WI-FI
#include <WiFi.h>

// crear un objeto con el ssid y la constraseña del host
const char* ssid = "Mi Ada";
const char* password = "telefono4ada";

// elige el puerto 80 para el servidor wifi
WiFiServer server(80);

// Variable tpara guardar el HTTP request
String header;

//Variables auxiliares para guardar el estado del output
String output26State = "off";
String output27State = "off";

// asigna los outputs a los pines
const int output26 = 26;
const int output27 = 27;

// miramos el tiempo actual que ha pasado desde empezar el programa
unsigned long currentTime = millis();
//iramos el tiempo que ha pasado desde empezar el programa
unsigned long previousTime = 0; 
// Define el timeout en milisegundos, es decir 2000ms = 2s 
const long timeoutTime = 2000;

void setup() {
  Serial.begin(115200); //inicialice una comunicación en serie a una velocidad de transmisión de 115200
  // Initializa los pines
  pinMode(output26, OUTPUT);
  pinMode(output27, OUTPUT);
  // y ponlos a nivel bajo
  digitalWrite(output26, LOW);
  digitalWrite(output27, LOW);

  // Conecta el WI-FI  con el SSID y la contraseña
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  // Muestra por pantalla la IP local y inicializa el servidor
  Serial.println("");
  Serial.println("WiFi connected.");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  server.begin();
}

void loop(){
  WiFiClient client = server.available();   // Mira la variable para después poder mirar ciertos parámetros

  if (client) {                             // si se conecta un nuevo cliente al servidor
    currentTime = millis();
    previousTime = currentTime;
    Serial.println("New Client.");          // muestra el mensaje por la terminal 
    String currentLine = "";                // string para poder tener los datos entrantes del cliente 
    while (client.connected() && currentTime - previousTime <= timeoutTime) {  // bucle mientras el cliente está conectado 
      currentTime = millis();
      if (client.available()) {             // si sigue habiendo bytes para leer del cliente
        char c = client.read();             // léelo 
        Serial.write(c);                    // y enseñalo por la terminal
        header += c;
        if (c == '\n') {                    // si el byte es un carácter de una nueva línea 
          // si la línea actual está vacia , tendrás 2 carácteres de nueva línea en fila eso significa que es el final del HTTP Request del cliente
          // Entonces responde de esta manera
          if (currentLine.length() == 0) {
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println("Connection: close");
            client.println();
            
            // enciende y apaga los GPIOs 
            if (header.indexOf("GET /26/on") >= 0) {
              Serial.println("GPIO 26 on");
              output26State = "on";
              digitalWrite(output26, HIGH);
            } else if (header.indexOf("GET /26/off") >= 0) {
              Serial.println("GPIO 26 off");
              output26State = "off";
              digitalWrite(output26, LOW);
            } else if (header.indexOf("GET /27/on") >= 0) {
              Serial.println("GPIO 27 on");
              output27State = "on";
              digitalWrite(output27, HIGH);
            } else if (header.indexOf("GET /27/off") >= 0) {
              Serial.println("GPIO 27 off");
              output27State = "off";
              digitalWrite(output27, LOW);
            }
            
            // código para la página web en HTML
            client.println("<!DOCTYPE html><html>");
            client.println("<head><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">");
            client.println("<link rel=\"icon\" href=\"data:,\">");
            // CSS to style the on/off buttons 
            // Feel free to change the background-color and font-size attributes to fit your preferences
            client.println("<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}");
            client.println(".button { background-color: #4CAF50; border: none; color: white; padding: 16px 40px;");
            client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
            client.println(".button2 {background-color: #555555;}</style></head>");
            
            // Cabecera de la página web
            client.println("<body><h1>ESP32 Web Server</h1>");
            
            // estado del display (web) y de los botones ON/OFF para el pin 26   
            client.println("<p>GPIO 26 - State " + output26State + "</p>");
            // si el output26State está en off, aparece ON en el botón
            if (output26State=="off") {
              client.println("<p><a href=\"/26/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/26/off\"><button class=\"button button2\">OFF</button></a></p>");
            } 
               
          // estado del display (web) y de los botones ON/OFF para el pin 27 
            client.println("<p>GPIO 27 - State " + output27State + "</p>");
        // si el output27State está en off, aparece ON en el botón     
            if (output27State=="off") {
              client.println("<p><a href=\"/27/on\"><button class=\"button\">ON</button></a></p>");
            } else {
              client.println("<p><a href=\"/27/off\"><button class=\"button button2\">OFF</button></a></p>");
            }
            client.println("</body></html>");
            
            // La respuesta HTTP acaba con otra línea en blanco 
            client.println();
            // se rompe el loop
            break;
          } else { //si sale una nueva línea, entonces haz un clear a la que tienes 
            currentLine = "";
          }
        } else if (c != '\r') {  // si solo tienes un carácter de acarreo añadelo a la línea actual 
          currentLine += c;      
        }
      }
    }
    // limpia la variable de la cabecera 
    header = "";
    // cierra conexión 
    client.stop();
       Serial.println("Client disconnected."); //enseñalo por la terminal 
    Serial.println("");
  }
}
```

## Cómo Ejecutar (Versión en Español)

1. **Instalar PlatformIO**
   - Instala [VS Code](https://code.visualstudio.com/)
   - Instala la [extensión PlatformIO](https://platformio.org/install/ide?install=vscode)

2. **Crear un Nuevo Proyecto**
   - Abre PlatformIO en VS Code
   - Crea un nuevo proyecto y selecciona tu placa (ejemplo: ESP32 Dev Module)
   - Elige el **framework Arduino**

3. **Agregar el Código**
   - Reemplaza el contenido de `src/main.cpp` con el código proporcionado arriba

4. **Conectar el Hardware**
   - Conecta la placa ESP32 a tu computadora mediante USB
   - Conecta **LED1** al GPIO 26 con una resistencia (470Ω–1kΩ) en serie
   - Conecta **LED2** al GPIO 27 con una resistencia (470Ω–1kΩ) en serie

5. **Compilar y Subir**
   - Haz clic en **Build (✓)** para compilar el código
   - Haz clic en **Upload (→)** para cargar el código en tu ESP32

6. **Observar el Resultado**
   - Abre un navegador y navega a la IP del ESP32 que aparece en el Monitor Serial
   - Usa los botones ON/OFF de la página web para controlar LED1 y LED2

### Recursos
- **Video demostrativo en español:** [Ver video](assets/practica3extra.mp4) 
- **Documentación del ESP32:** [Espressif ESP32](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/index.html)  
- **Documentación de Wi-Fi en ESP32:** [ESP32 Wi-Fi API](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/network/esp_wifi.html)  
- **Referencia de la librería Wi-Fi en Arduino:** [Arduino WiFi](https://www.arduino.cc/en/Reference/WiFi)  


