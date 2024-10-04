# CP5-EDGE


## Descrição

Este projeto utiliza um ESP32 e os sensores DHT22 e LDR para monitorar a temperatura, umidade e luminosidade. Os dados são lidos e podem ser visualizados através do Monitor Serial na IDE do Arduino.

## Componentes

### 1. ESP32
- **Descrição**: Microcontrolador que possui Wi-Fi e Bluetooth integrados, ideal para projetos de IoT.

### 2. DHT11
- **Descrição**: Sensor de temperatura e umidade digital. Ele fornece leituras precisas de temperatura e umidade em um único módulo.

### 3. LDR
- **Descrição**: Sensor de luz que varia sua resistência com a intensidade da luz. É utilizado para medir a luminosidade.

### 4. LED (Vermelho)
- **Descrição**: Um LED que pode ser usado como indicador visual de status.

### 5. Resistor de 1kΩ (R1 e R2)
- **Descrição**: Usado para limitar a corrente elétrica que passa pelo LED e proteger o circuito.

## Código

Utilize o código fornecido anteriormente para fazer a leitura dos sensores e visualizar os dados no Monitor Serial.

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

// Definições de pinos
#define DHTPIN 4       // Pino onde o DHT11 está conectado
#define DHTTYPE DHT11  // Define o tipo do sensor DHT

// Configurações WiFi e MQTT
const char* ssid = "5.G_RDLM24";
const char* password = "Daniroluma722";
const char* mqtt_server = "192.168.0.100";

WiFiClient espClient;
PubSubClient client(espClient);
DHT dht(DHTPIN, DHTTYPE);

// Variáveis para o LDR
const int ldrPin = 34;

void setup() {
  Serial.begin(115200);
  dht.begin();
  
  setup_wifi();
  client.setServer(mqtt_server, 1883);

  pinMode(ldrPin, INPUT);  // Configura o pino do LDR como entrada
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
  
  // Leitura dos sensores
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  int ldrValue = analogRead(ldrPin);

  if (isnan(h) || isnan(t)) {
    Serial.println("Erro ao ler do DHT11");
    return;
  }

  // Publica temperatura, umidade e luminosidade no servidor MQTT
  String payload = "Temperatura: " + String(t) + "C, Umidade: " + String(h) + "%, Luminosidade: " + String(ldrValue);
  client.publish("sensor/dados", payload.c_str());

  delay(2000);  // Envia os dados a cada 2 segundos
}

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Conectando-se a ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi conectado.");
  Serial.println("IP obtido: ");
  Serial.println(WiFi.localIP());
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Tentando conectar ao servidor MQTT...");
    if (client.connect("ESP32Client")) {
      Serial.println("conectado");
      client.subscribe("sensor/comando");
    } else {
      Serial.print("falha, rc=");
      Serial.print(client.state());
      Serial.println(" tentando novamente em 5 segundos");
      delay(5000);
    }
  }
}
```
## Integrantes
**Guilherme Lunghini RM556892
**Luan Ramos RM558537
**Matheus Bortolotto RM555189
**Matheus Ricciotti RM556930


