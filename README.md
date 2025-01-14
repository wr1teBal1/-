# 许恒睿——嵌入式赛道仓库

## TASK 1
环境配置成功，编译并烧录成功如下图
![4c60c297f75203a9a72f95b8b0b5a9b](https://github.com/user-attachments/assets/7bc869df-b99c-4425-bcfc-c58a689aac33)

烧录效果图

https://github.com/user-attachments/assets/d8208729-484a-4431-81bb-733bfd2f66a7

源码如下，运用示例中的basics，重新定义esp32 cam的led引脚pin 4即可

```c++
#define LED_BUILTIN 4
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT);
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);  // turn the LED on (HIGH is the voltage level)
  delay(1000);                      // wait for a second
  digitalWrite(LED_BUILTIN, LOW);   // turn the LED off by making the voltage LOW
  delay(1000);                      // wait for a second
}
```
## TASK2.0
学习串⼝协议（ USART） ，向电脑发送hello world。
```c++
void setup() {
  // 初始化串口通信，波特率为115200
  Serial.begin(115200);
  delay(1000); // 延时1秒以确保串口正常启动
}

void loop() {
  // 每秒发送一次 "Hello World"
  Serial.println("Hello World");
  delay(1000);
}

```
下面是在串口程序中检测到的数据
![58ec2ce25ecebea9f8eee4dcc697db4](https://github.com/user-attachments/assets/63007143-a873-4e5f-b8d7-312799028533)


### TASK 2.1
学习esp32单⽚机的WiFi模块，连接⾃⼰的⼿机热点

我的链接热点源码是学长亲手在csdn淘的qwq
用源代码的话，我的单片机不知道为什么连不上，下面是参考文献连接和源码
https://wenku.csdn.net/answer/0565459f313c45a7a0f82e722cd75546
```c++
#include <WiFi.h>
#include <WebServer.h>

const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";
IPAddress localIP(192, 168, 1, 100); // ESP32-CAM的IP地址
WebServer server(80);

void handleCommand() {
  String command = server.arg("command");

  if (command == "on") {
    // 执行打开操作
    Serial.println("Turning on...");
  } else if (command == "off") {
    // 执行关闭操作
    Serial.println("Turning off...");
  } else {
    // 无效的命令
    Serial.println("Invalid command");
  }

  server.send(200, "text/plain", "OK");
}

void setup() {
  Serial.begin(115200);
  delay(1000);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }

  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  server.on("/command", handleCommand);
  server.begin();
}

void loop() {
  server.handleClient();
}
```
连接成功并发送自己的ip地址

![431fce772d5f29eb40ac84951c8f4e8](https://github.com/user-attachments/assets/f90d215b-6b86-4e5a-a117-0c67d3023cfd)

手机显示已连接

![image](https://github.com/user-attachments/assets/62de9f1d-4a6f-41e0-8244-a4b18be7f92b)


### TASK 2.2
通过阿⾥云物联⽹平台（需要注册，但是对学⽣免费），从云端发信息到已经连接WiFi的esp32单⽚
机，完成云端到mcu的数据流动，可以使⽤点灯等形式展现。

首先在setup中设置灯引脚

```c++
  pinMode(4,OUTPUT);

```

接受阿里云平台topic并解析

```c++
void onMqttMessage(int messageSize) {
  // we received a message, print out the topic and contents
  Serial.print("Received a message with topic '");
  Serial.print(mqttClient.messageTopic());
  Serial.print("', duplicate = ");
  Serial.print(mqttClient.messageDup() ? "true" : "false");
  Serial.print(", QoS = ");
  Serial.print(mqttClient.messageQoS());
  Serial.print(", retained = ");
  Serial.print(mqttClient.messageRetain() ? "true" : "false");
  Serial.print("', length ");
  Serial.print(messageSize);
  Serial.println(" bytes:");

  // use the Stream interface to print the contents
  while (mqttClient.available()) {
    char inChar =(char)mqttClient.read();
    inputString +=inChar;
    if(inputString.length()==messageSize) {

      DynamicJsonDocument json_msg(1024);
      DynamicJsonDocument json_item(1024);
      DynamicJsonDocument json_value(1024);

      deserializeJson(json_msg,inputString);

      String items = json_msg["items"];

      deserializeJson(json_item,items);
      String led = json_item["led"];

      deserializeJson(json_value,led);
      bool value = json_value["value"];
```

解析命令,给予实现并打印

```c++
if(value ==0) {
        //关
        Serial.println("off");
        digitalWrite(4,LOW);
      } else {
        //开
        Serial.println("on");
        digitalWrite(4,HIGH);
      }
```
![027fa2f7c1dd35323a853e08b65827f](https://github.com/user-attachments/assets/1247db81-2d40-40e1-9384-7ae749678db3)
![6d64f3f2f37dd68068b9c8392a1d1ab](https://github.com/user-attachments/assets/a732fe59-ff92-42da-a487-1373e64d8d0f)

最后清空inputString

```c++
inputString = "";
```
最终源码展示
```c++
#include <WiFi.h>
#include <WebServer.h>
#include <ArduinoMqttClient.h>
#include <ArduinoJson.h>

const char* ssid = "MI8";
const char* password = "88888888";
IPAddress localIP(192, 168, 4, 1); // ESP32-CAM的IP地址
WebServer server(80);

WiFiClient wifiClient;
MqttClient mqttClient(wifiClient);


const char broker[]    = "iot-06z00fam12dey3c.mqtt.iothub.aliyuncs.com";
int        port        = 1883;

const char inTopic[]   = "/sys/k28xr7bv6Qh/esp32cam/thing/service/property/set";
const char outTopic[]  = "/sys/k28xr7bv6Qh/esp32cam/thing/event/property/post";

const long interval = 10000;
unsigned long previousMillis = 0;

int count = 0;

String inputString ="";

void setup() {
  Serial.begin(115200);
  delay(1000);

  pinMode(4,OUTPUT);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }

  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  mqttClient.setId("k28xr7bv6Qh.esp32cam|securemode=2,signmethod=hmacsha256,timestamp=1736782630523|");                    //mqtt 连接客户端id
  mqttClient.setUsernamePassword("esp32cam&k28xr7bv6Qh", "3d59e1bf7add299ac391cafef98e4e225f2eaafa85eafe4690ef8e202fc9bc41");        //mqtt 连接用户名、密码

  Serial.print("Attempting to connect to the MQTT broker: ");
  Serial.println(broker);
   if (!mqttClient.connect(broker, port)) {
    Serial.print("MQTT connection failed! Error code = ");
    Serial.println(mqttClient.connectError());

    while (1);
  }

  Serial.println("You're connected to the MQTT broker!");
  Serial.println();

  // set the message receive callback
  mqttClient.onMessage(onMqttMessage);

  Serial.print("Subscribing to topic: ");
  Serial.println(inTopic);
  Serial.println();

  int subscribeQos = 1;

  mqttClient.subscribe(inTopic, subscribeQos);

  // topics can be unsubscribed using:
  // mqttClient.unsubscribe(inTopic);

  Serial.print("Waiting for messages on topic: ");
  Serial.println(inTopic);
  Serial.println();

  server.begin();

}

void loop() {
  server.handleClient();
   mqttClient.poll();


  unsigned long currentMillis = millis();

  if (currentMillis - previousMillis >= interval) {
    // save the last time a message was sent
    previousMillis = currentMillis;

    String payload;

//    payload = "{\"params\":{\"led\":1},\"version\":\"1.0.0\"}";
  
  

    DynamicJsonDocument json_msg(512);
    DynamicJsonDocument json_data(512);


    json_msg["params"] = json_data;
    json_msg["version"] = "1.0.0";


    serializeJson(json_msg,payload);

    Serial.print("Sending message to topic: ");
    Serial.println(outTopic);
    Serial.println(payload);


    bool retained = false;
    int qos = 1;
    bool dup = false;

    mqttClient.beginMessage(outTopic, payload.length(), retained, qos, dup);
    mqttClient.print(payload);
    mqttClient.endMessage();

    Serial.println();

    count++;
  }
}
void onMqttMessage(int messageSize) {
  // we received a message, print out the topic and contents
  Serial.print("Received a message with topic '");
  Serial.print(mqttClient.messageTopic());
  Serial.print("', duplicate = ");
  Serial.print(mqttClient.messageDup() ? "true" : "false");
  Serial.print(", QoS = ");
  Serial.print(mqttClient.messageQoS());
  Serial.print(", retained = ");
  Serial.print(mqttClient.messageRetain() ? "true" : "false");
  Serial.print("', length ");
  Serial.print(messageSize);
  Serial.println(" bytes:");

  // use the Stream interface to print the contents
  while (mqttClient.available()) {
    char inChar =(char)mqttClient.read();
    inputString +=inChar;
    if(inputString.length()==messageSize) {

      DynamicJsonDocument json_msg(1024);
      DynamicJsonDocument json_item(1024);
      DynamicJsonDocument json_value(1024);

      deserializeJson(json_msg,inputString);

      String items = json_msg["items"];

      deserializeJson(json_item,items);
      String led = json_item["led"];

      deserializeJson(json_value,led);
      bool value = json_value["value"];

      if(value ==0) {
        //关
        Serial.println("off");
        digitalWrite(4,LOW);
      } else {
        //开
        Serial.println("on");
        digitalWrite(4,HIGH);
      }
      inputString="";
    }


  }
  Serial.println();

  Serial.println();
}
```
### TASK 2.2.1
拓展1:使⽤Android Studio，搭建⼀个⼿机软件，在⼿机软件界⾯完成点灯等任务。
### TASK 2.2.2
拓展2:如果你不愿意使⽤软件进⾏展⽰，也可以选择搭建⼀个web，进⾏展⽰（阿⾥云物联⽹平台有免
费对学⽣开放的简单web搭建，也可以选择⼿搓）
### TASK 2.2.3
拓展3:使⽤阿⾥云物联⽹平台，从mcu往云端发送数据，并且在⼿机软件或web界⾯显⽰。（可以⾃⼰
设定⼀个数据，也可以使⽤mcu读取传感器数据）
### TASK 2.2.4
拓展4:观察你⼿⾥⾯的ESP32-CAM，你会发现，他有⼀个摄像头。请将图像数据回传到云端，并且在
⾃⼰的⼿机软件或者web界⾯进⾏展⽰
