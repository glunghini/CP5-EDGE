# CP5-EDGE

Link do projeto

[https://wokwi.com/projects/410679081435333633](https://wokwi.com/projects/410858903870433281
 )

![image](https://github.com/user-attachments/assets/ad44992a-fe0c-467d-a6d8-0af709d91d0c)


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
 

const char* default_SSID = "FIAP-IBM"; // Nome da rede Wi-Fi
const char* default_PASSWORD = "Challenge@24!"; // Senha da rede Wi-Fi
const char* default_BROKER_MQTT = "18.208.160.16"; // IP do Broker MQTT
const int default_BROKER_PORT = 1883; // Porta do Broker MQTT
const char* default_TOPICO_SUBSCRIBE = "/TEF/device042/cmd"; // Tópico MQTT de escuta
const char* default_TOPICO_PUBLISH_1 = "/TEF/device042/attrs"; // Tópico MQTT de envio de informações para Broker
const char* default_TOPICO_PUBLISH_2 = "/TEF/device042/attrs/p"; // Tópico MQTT de envio de informações para Broker
const char* default_ID_MQTT = "fiware_042"; // ID MQTT
const int default_D4 = 2; // Pino do LED onboard
const int ldrPin = 34; // Pino do LDR para leitura analógica
#define DHTPIN 4 // Pino de dados do DHT11
#define DHTTYPE DHT11 // Tipo de sensor DHT11

char* SSID = const_cast<char*>(default_SSID);
char* PASSWORD = const_cast<char*>(default_PASSWORD);
char* BROKER_MQTT = const_cast<char*>(default_BROKER_MQTT);
int BROKER_PORT = default_BROKER_PORT;
char* TOPICO_SUBSCRIBE = const_cast<char*>(default_TOPICO_SUBSCRIBE);
char* TOPICO_PUBLISH_1 = const_cast<char*>(default_TOPICO_PUBLISH_1);
char* TOPICO_PUBLISH_2 = const_cast<char*>(default_TOPICO_PUBLISH_2);
char* ID_MQTT = const_cast<char*>(default_ID_MQTT);
int D4 = default_D4;
 
WiFiClient espClient;
PubSubClient MQTT(espClient);
DHT dht(DHTPIN, DHTTYPE);  
char EstadoSaida = '0';
 
void initSerial() {
    Serial.begin(115200);
}
 
void initWiFi() {
    delay(10);
    Serial.println("------Conexao WI-FI------");
    Serial.print("Conectando-se na rede: ");
    Serial.println(SSID);
    Serial.println("Aguarde");
    reconectWiFi();
}
 
void initMQTT() {
    MQTT.setServer(BROKER_MQTT, BROKER_PORT);
    MQTT.setCallback(mqtt_callback);
}
 
void setup() {
    InitOutput();
    initSerial();
    dht.begin(); 
    pinMode(ldrPin, INPUT); 
    initWiFi();
    initMQTT();
    delay(5000);
    MQTT.publish(TOPICO_PUBLISH_1, "s|on");
}
 
void loop() {
    VerificaConexoesWiFIEMQTT();
    EnviaEstadoOutputMQTT();
    handleSensores();
    MQTT.loop();
}
 
void reconectWiFi() {
    if (WiFi.status() == WL_CONNECTED)
        return;
    WiFi.begin(SSID, PASSWORD);
    while (WiFi.status() != WL_CONNECTED) {
        delay(100);
        Serial.print(".");
    }
    Serial.println();
    Serial.println("Conectado com sucesso na rede ");
    Serial.print(SSID);
    Serial.println("IP obtido: ");
    Serial.println(WiFi.localIP());
 
    digitalWrite(D4, LOW);
}
 
void mqtt_callback(char* topic, byte* payload, unsigned int length) {
    String msg;
    for (int i = 0; i < length; i++) {
        char c = (char)payload[i];
        msg += c;
    }
    Serial.print("- Mensagem recebida: ");
    Serial.println(msg);
 
    String onTopic = String("device001@on|");
    String offTopic = String("device001@off|");
 
    
    if (msg.equals(onTopic)) {
        digitalWrite(D4, HIGH);
        EstadoSaida = '1';
    }
 
    if (msg.equals(offTopic)) {
        digitalWrite(D4, LOW);
        EstadoSaida = '0';
    }
}
 
void VerificaConexoesWiFIEMQTT() {
    if (!MQTT.connected())
        reconnectMQTT();
    reconectWiFi();
}
 
void EnviaEstadoOutputMQTT() {
    if (EstadoSaida == '1') {
        MQTT.publish(TOPICO_PUBLISH_1, "s|on");
        Serial.println("- Led Ligado");
    }
 
    if (EstadoSaida == '0') {
        MQTT.publish(TOPICO_PUBLISH_1, "s|off");
        Serial.println("- Led Desligado");
    }
    Serial.println("- Estado do LED onboard enviado ao broker!");
    delay(1000);
}
 
void InitOutput() {
    pinMode(D4, OUTPUT);
    digitalWrite(D4, HIGH);
    boolean toggle = false;
 
    for (int i = 0; i <= 10; i++) {
        toggle = !toggle;
        digitalWrite(D4, toggle);
        delay(200);
    }
}
 
void reconnectMQTT() {
    while (!MQTT.connected()) {
        Serial.print("* Tentando se conectar ao Broker MQTT: ");
        Serial.println(BROKER_MQTT);
        if (MQTT.connect(ID_MQTT)) {
            Serial.println("Conectado com sucesso ao broker MQTT!");
            MQTT.subscribe(TOPICO_SUBSCRIBE);
        } else {
            Serial.println("Falha ao reconectar no broker.");
            Serial.println("Haverá nova tentativa de conexão em 2s");
            delay(2000);
        }
    }
}
 
void handleSensores() {
    
    float humidity = dht.readHumidity();
    float temperature = dht.readTemperature();
 
    
    int ldrValue = analogRead(ldrPin);
    int luminosity = map(ldrValue, 0, 4095, 0, 100);
 
    
    if (isnan(humidity) || isnan(temperature)) {
        Serial.println("Falha ao ler o sensor DHT!");
        return;
    }
 
    
    String sensorData = "Temperatura: " + String(temperature) + "C, " +
                        "Umidade: " + String(humidity) + "%, " +
                        "Luminosidade: " + String(luminosity);
 
    Serial.println(sensorData);
 
    MQTT.publish(TOPICO_PUBLISH_2, sensorData.c_str());
}
 
```
## Integrantes

Guilherme Lunghini RM556892

Luan Ramos RM558537

Matheus Bortolotto RM555189

Matheus Ricciotti RM556930


