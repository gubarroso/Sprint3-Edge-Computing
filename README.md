Gustavo Garcia Silva RM:562078
Gustavo Oliveira Barroso RM:565705
João Marcelo Diniz Vespa RM:564038
Nicolas Santana Gará RM:561461

------------------------------------------------------------------------------------------------

*link do wokwi*
https://wokwi.com/projects/441914310935802881

*link do thinkspeak*
https://thingspeak.mathworks.com/channels/3073129

------------------------------------------------------------------------------------------------

*Descrição do Projeto*

O projeto tem como objetivo monitorar o rendimento físico de atletas em tempo real, utilizando Edge Computing para processar dados localmente e enviar informações para visualização em dashboard.
Como os sensores reais de rendimento não estão disponíveis, utilizamos uma simulação no Wokwi, onde:

DHT22 representa temperatura e umidade, simulando frequência cardíaca e velocidade.
Potenciômetro simula a distância percorrida pelo atleta.
LED vermelho indica o envio de dados com sucesso (apenas local).

O sistema permite análises de desempenho, registro de dados históricos e alertas rápidos, integrando IoT e processamento local para maior eficiência.

*Explicação do Projeto*

Sensores: O ESP32 recebe dados do DHT22 (temperatura → frequência cardíaca, umidade → velocidade) e do potenciômetro (distância percorrida).
Processamento Local: O microcontrolador processa os dados antes de enviá-los, aplicando Edge Computing para respostas rápidas.
Indicação Local: O LED vermelho pisca quando os dados são enviados com sucesso.
Envio para Dashboard: Os dados processados são enviados ao ThingSpeak, possibilitando visualização remota em tempo real.

*Recursos Necessários*

Hardware (simulado ou real):
  ESP32
  DHT22
  Potenciômetro
  LED vermelho

Software:
  Arduino IDE / Wokwi
  Bibliotecas: DHT.h, WiFi.h, HTTPClient.h
  Conta no ThingSpeak para criar canal de monitoramento

ThingSpeak Configuração:
  Channel ID: (informar)
  Write API Key: Z0MBJDI8XE2JAXWA

Fields sugeridos:
  Field 1 → Frequência cardíaca
  Field 2 → Velocidade
  Field 3 → Distância percorrida

*Instruções de Uso*

  Montagem do Hardware / Simulação:
    Conectar DHT22 ao ESP32 (ex.: GPIO15).
    Conectar potenciômetro ao ESP32.
    Conectar LED vermelho a uma porta digital para indicação de envio.

  Programação do ESP32:
    Abrir código_esp32.ino na Arduino IDE.
    Configurar rede Wi-Fi e API Key do ThingSpeak.
    Fazer upload do código para o ESP32.

  Execução e Monitoramento:
    O ESP32 lê os sensores e processa os dados localmente.
    O LED vermelho pisca quando os dados são enviados com sucesso.
    Os dados são enviados aos campos correspondentes do ThingSpeak para visualização.

------------------------------------------------------------------------------------------------

*Código Fonte do Wokwi*

#include <WiFi.h>
#include <HTTPClient.h>
#include "DHT.h"

#define DHTPIN 15     
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

const char* ssid = "Wokwi-GUEST";  // WiFi padrão do Wokwi
const char* password = "";

String apiKey = "Z0MBJDI8XE2JAXWA";    // Sua Write API Key
String channelID = "3073129";          // Seu Channel ID

const char* server = "http://api.thingspeak.com/update";

int potPin = 34;   // Potenciômetro no pino 34
int ledPin = 2;    // LED no pino 2 (LED onboard ou externo)

void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);
  dht.begin();

  WiFi.begin(ssid, password);
  Serial.print("Conectando ao WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi conectado!");
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    float temp = dht.readTemperature();   // Representa frequência cardíaca
    float hum = dht.readHumidity();       // Representa velocidade
    int potValue = analogRead(potPin);    // Representa distância
    float distancia = map(potValue, 0, 4095, 0, 1000); // Escala 0–1000m

    if (isnan(temp) || isnan(hum)) {
      Serial.println("Falha ao ler do DHT!");
      return;
    }

    HTTPClient http;
    String url = server;
    url += "?api_key=" + apiKey;
    url += "&field1=" + String(temp);       // Frequência cardíaca
    url += "&field2=" + String(hum);        // Velocidade
    url += "&field3=" + String(distancia);  // Distância

    http.begin(url);
    int httpCode = http.GET();
    if (httpCode > 0) {
      Serial.println("Dados enviados com sucesso!");
      Serial.println("HR (temp) = " + String(temp) + 
                     " | Vel (hum) = " + String(hum) + 
                     " | Dist (pot) = " + String(distancia));
      digitalWrite(ledPin, HIGH);
      delay(200);
      digitalWrite(ledPin, LOW);
    } else {
      Serial.println("Erro ao enviar: " + String(httpCode));
    }
    http.end();
  } else {
    Serial.println("WiFi desconectado!");
  }

  delay(20000); // ThingSpeak aceita no máx. 1 req a cada 15s
}

<img width="494" height="420" alt="image" src="https://github.com/user-attachments/assets/9cfaf461-6728-4809-abb1-1c6c04e59bd0" />



  Visualização dos Dados:
    Acessar o canal do ThingSpeak para monitorar gráficos de frequência cardíaca, velocidade e distância percorrida.
