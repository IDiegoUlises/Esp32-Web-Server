# Esp32 Web Server

```c++
#include <WiFi.h>

//Iniciamos servidor web en el puerto 80
WiFiServer server(80);

const char* ssid     = "Wifi hogar";
const char* password = "panconpalta";

int conexion = 0; //guarda el tiempo de conexion
String header; // variable para guardar el HTTP request
String estadoSalida = "off";
const int pinLed = 2;

//Codigo html
String pagina = "<!DOCTYPE html>"
                "<html>"
                "<head>"
                "<meta charset='utf-8' />"
                "<title>Servidor Web ESP32</title>"
                "</head>"
                "<body>"
                "<center>"
                "<h1>Servidor Web ESP32</h1>"
                "<p><a href='/on'><button style='height:50px;width:100px'>ON</button></a></p>"
                "<p><a href='/off'><button style='height:50px;width:100px'>OFF</button></a></p>"
                "</center>"
                "</body>"
                "</html>";


void setup() 
{
  Serial.begin(9600);
  Serial.println("");

  pinMode(pinLed, OUTPUT);
  digitalWrite(pinLed, LOW);

  // Conexión WIFI
  WiFi.begin(ssid, password);
  
  //Cuenta hasta 50 si no se puede conectar lo cancela
  while (WiFi.status() != WL_CONNECTED and conexion < 50) 
  {
    ++conexion;
    delay(500);
    Serial.print(".");
  }
  if (conexion < 50)
  {
    Serial.println("");
    Serial.println("WiFi conectado");
    Serial.println(WiFi.localIP());
    server.begin(); //iniciamos el servidor
  }
  else 
  {
    Serial.println("");
    Serial.println("Error de conexion");
  }
}


void loop() 
{
  WiFiClient client = server.available();   //escucha a los clientes entrantes

  if (client) 
  {   
    // Si se conecta un nuevo cliente
    Serial.println("New Client.");          //
    String currentLine = "";                //
    while (client.connected())
    { 
      // loop mientras el cliente está conectado
      if (client.available()) 
      {
       // si hay bytes para leer desde el cliente
        char c = client.read();             // lee un byte
        Serial.write(c);                    // imprime ese byte en el monitor serial
        header += c;
        if (c == '\n') 
        {
          // si el byte es un caracter de salto de linea
          // si la nueva linea está en blanco significa que es el fin del
          // HTTP request del cliente, entonces respondemos:
          if (currentLine.length() == 0) {
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println("Connection: close");
            client.println();

            // enciende y apaga el GPIO
            if (header.indexOf("GET /on") >= 0) {
              Serial.println("GPIO on");
              estadoSalida = "on";
              digitalWrite(pinLed, HIGH);
            } else if (header.indexOf("GET /off") >= 0) {
              Serial.println("GPIO off");
              estadoSalida = "off";
              digitalWrite(pinLed, LOW);
            }

            // Muestra la página web
            client.println(pagina);

            // la respuesta HTTP temina con una linea en blanco
            client.println();
            break;
          }
          else 
          { 
            // si tenemos una nueva linea limpiamos currentLine
            currentLine = "";
          }
        } 
        else if (c != '\r') 
        { 
          // si C es distinto al caracter de retorno de carro
          currentLine += c;      // lo agrega al final de currentLine
        }
      }
    }
    // Limpiamos la variable header
    header = "";
    // Cerramos la conexión
    client.stop();
    Serial.println("Client disconnected.");
    Serial.println("");
  }
}
```
