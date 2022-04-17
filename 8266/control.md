
```c
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#define JDQ 16 //D0 空调
#define light 5 //D1  灯
#define fire 4 //D2空调暖风
#define white 0 //D3 白板
#define sound 2 //D4 蜂鸣器
const char* ssid = "Ustl";
//连接的路由器的名字
const char* password = "ustl@123";
//连接的路由器的密码
const char* mqtt_server = "192.168.1.106";
//服务器的地址 
const int port=1883;
//服务器端口号    
const char* topic_name = "smart-control";
//订阅的主题
const char* client_id = "smart-control-byxj";
//尽量保持唯一，相同的id连接会被替代
WiFiClient espClient;
PubSubClient client(espClient);
//初始化WIFI
void setup_wifi() {
  //自动连WIFI接入网络
  delay(10);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print("...");
  }
}
void callback(char* topic, byte* payload, unsigned int length) {
  String callMsg = "";
  Serial.print("Message arrived [");
  Serial.print(topic);
  // 打印主题信息
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    callMsg += char(payload[i]);
  }
  Serial.println(callMsg);
  // 冷风
  if(callMsg.equals("cold_air_false")) {
    digitalWrite(JDQ, LOW);
    // Turn the LED on (Note that LOW is the voltage level
  }
  if(callMsg.equals("cold_air_true")) {
    digitalWrite(JDQ, HIGH);
    // Turn the LED off by making the voltage HIGH
  }

 //日光灯
   if(callMsg.equals("light_false")) {
    digitalWrite(light, LOW);
    // Turn the LED on (Note that LOW is the voltage level
  }
 
  if(callMsg.equals("light_true")) {
    digitalWrite(light, HIGH);
    // Turn the LED off by making the voltage HIGH
  }


   //暖风
   if(callMsg.equals("fire_air_false")) {
    digitalWrite(fire, LOW);
    // Turn the LED on (Note that LOW is the voltage level
  }
 
  if(callMsg.equals("fire_air_true")) {
    digitalWrite(fire, HIGH);
    // Turn the LED off by making the voltage HIGH
  }


     //白板
   if(callMsg.equals("white_bord_false")) {
    digitalWrite(white, LOW);
    // Turn the LED on (Note that LOW is the voltage level
  }
 
  if(callMsg.equals("white_bord_true")) {
    digitalWrite(white, HIGH);
    // Turn the LED off by making the voltage HIGH
  }

     //蜂鸣器
   if(callMsg.equals("sound_true")) {
    digitalWrite(sound, LOW);
    delay(1000);
    digitalWrite(sound, HIGH);
    // Turn the LED on (Note that LOW is the voltage level
  }
 
  if(callMsg.equals("sound_false")) {
    digitalWrite(sound, HIGH);
    // Turn the LED off by making the voltage HIGH
  }
}
void reconnect() {
  //等待，直到连接上服务器
  while (!client.connected()) {
    //如果没有连接上
    if (client.connect(client_id)) {
      //接入时的用户名，尽量取一个很不常用的用户名
      client.subscribe(topic_name);
      //接收外来的数据时的intopic
      Serial.println("connect success");
    } else {
      Serial.print("failed, rc=");
      //连接失败
      Serial.print(client.state());
      //重新连接
      Serial.println(" try again in 5 seconds");
      //延时5秒后重新连接
      delay(5000);
    }
  }
}
void setup() {
  //初始化程序，只运行一遍
  Serial.begin(115200);
  //设置串口波特率（与烧写用波特率不是一个概念）
  //pinMode(LED_BUILTIN, OUTPUT);
  pinMode(JDQ, OUTPUT);
   pinMode(light, OUTPUT);
    pinMode(fire, OUTPUT);
     pinMode(white, OUTPUT);
      pinMode(sound, OUTPUT);
  // Initialize the LED_BUILTIN pin as an output
  setup_wifi();
  //自动连WIFI接入网络
  client.setServer(mqtt_server, port);
  //端口号
  client.setCallback(callback);
  //用于接收服务器接收的数据

}
void loop() {
  //主循环
  String msg = "";
  //用于存放
  reconnect();
  //确保连上服务器，否则一直等待。
  client.loop();
  //MUC接收数据的主循环函数。
  //digitalWrite(JDQ, HIGH);
  while (Serial.available() > 0) {
    msg += char(Serial.read());
    delay(2);
  }
  int msglen = msg.length();
  if (msglen > 0) {
    Serial.println(msg);
    char msgArr[msglen+1];
    msg.toCharArray(msgArr,msglen + 1);
    client.publish(topic_name,msgArr);
  }
}
```
