# Práctica 3: ejercicio extra

## Materiales
- ESP32
- 2 resistencias ( 470ohms hasta 1kohms)
- 2 LEDs
## Presentación:
En esta práctica, igual a las anteriores P3s, también se centra en el periférico de WI-FI de la ESP32. Pero 
a su vez se conecta a 2 LEDs que se encienden si en la página web se ha picado a ON y se apagan si se pica
a OFF. Cada LED tiene su respectivo boton en el web.

## Explicación del código (comentarios línea a línea) :

```
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
            
            // Cabezera de la página web
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
    // limpia la variable de la cabezera 
    header = "";
    // cierra conexión 
    client.stop();
       Serial.println("Client disconnected."); //enseñalo por la terminal 
    Serial.println("");
  }
}
```

Observar el vídeo para ver las salidas con más claridad.
