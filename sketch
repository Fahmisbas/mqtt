#include <ESP8266WiFi.h>
#include <PubSubClient.h>

const char* ssid = "Cisco16093";
const char* password = "123456789";
const char* mqtt_server = "192.168.1.143";

bool ledState = LOW;
long lastMsg = 0;

// tambahkan dua baris dibawah ini
WiFiClient espClient;
PubSubClient client(espClient);

void setup_wifi() {

  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  
  // Mencoba koneksi ke WiFi
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  // Jika koneksi WiFi berhasil maka akan menampilkan teks dibawah pada Serial.
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  Serial.print("payload ");
  Serial.println(payload[0]);
  
  /*
    Nilai ID perangkat menurut Warna LED : 
    
     ID_PERANGKAT | Warna LED
    --------------------------
         1        |    RED
         2        |    GREEN
         3        |    WHITE 
    */
  if (payload[0] == '3') {
    if (payload[2] == 'a') digitalWrite(D5, !ledState);   
    ledState = !ledState;
  }
}

void reconnect() {
  // Melakukan perulangan hingga NodeMCU berhasil terhubung dengan broker MQTT.
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    
    // Membuat client ID secara acak 
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
   
    // Percobaan koneksi ke Broker MQTT.
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      
      // Mencoba publish message ke Broker dengan topic "data" setelah terhubung  
      client.publish("data", "hello world");
      
      // Melakukan subscribe ke topik "command". 
      // Topik "command" berisikan perintah dari perangkat Android.
      client.subscribe("command");
      
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      
      // Delay 5 detik sebelum melakukan koneksi ulang.
      delay(5000);
    }
  }
}


void setup() {
  pinMode(D5, OUTPUT); // Menginisialisasikan pin D5 sebagai output
  
  Serial.begin(115200); // Set baud-rate komunikasi serial monitor sebesar 115200 bps.  
  setup_wifi(); // Memanggil fungsi setup_wifi() yang sudah dibuat sebelumnya
  
  // Print teks "connected" pada serial monitor jika koneksi ke WiFi berhasil.
  Serial.println("connected"); 
  
  // Menghubungkan NodeMCU kepada 
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);

}

void loop() {
    // Memanggil fungsi reconnect() jika perangkat belum terhubung dengan MQTT Broker. 
  if (!client.connected()) {
    reconnect();
  }
  
  client.loop();

  // Mendapatkan waktu hidup perangkat dalam satuan millisecond.
  long now = millis();
  
  // Jika selisih waktu sudah 500 millisecond, maka jalankan perintah didalam blok if. 
  if (now - lastMsg > 500) {
    lastMsg = now;

    // Membaca nilai tegangan dari LDR dalam rentang 0-1023.
    int sensorValue = analogRead(A0);   // read the input on analog pin 0

    // Melakukan konversi satuan hasil baca analog (0-1023) menjadi voltage (0-5V). 
    float voltage = sensorValue * (3.3 / 1023.0);   

    // konversi nilai float tengangan menjadi array Char.
    char buf[8];
    sprintf(buf, "%f", voltage);

    // Menggabungkan nilai tegangan LDR dengan ID pengenal Perangkat. 
    char messageDelivered[10];
     
    /*
    Nilai ID perangkat menurut Warna LED : 
    
     ID_PERANGKAT | Warna LED
    --------------------------
         1        |    RED
         2        |    GREEN
         3        |    WHITE 
    */
    strcpy(messageDelivered,"3");
    strcat(messageDelivered," ");
    strcat(messageDelivered,buf);
      
    // Mencetak message yang akan di publish ke broker pada serial monitor 
    Serial.print("Publish message: ");
    Serial.println(messageDelivered);
    
    // Publish message ke topic "data"
    client.publish("data", messageDelivered);
  }
}
