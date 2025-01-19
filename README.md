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

设置软件界面

文字与按钮显示（“关”按钮省略）
```java
  <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_marginTop="50dp">

        <TextView
            android:layout_width="254dp"
            android:layout_height="wrap_content"
            android:text="灯怎么灭了，点一下试试？"
            android:textColor="@color/black"
            android:textSize="17sp" />

        <Button
            android:id="@+id/btn_open"
            android:layout_width="100dp"
            android:layout_height="wrap_content"
            android:text="开"
            android:textColor="@color/black"
            android:textSize="17sp" />


    </LinearLayout>
```
![4e2c73801a71652942d01e52655ab0b](https://github.com/user-attachments/assets/7a3b3d36-c751-4efe-839a-15096aef22e8)


定义密钥，用户名及密码

```java
    private String productKey = "k28xr7bv6Qh";
    private String deviceName = "app_dev";

    private String deviceSecret = "89c41d57befdc89787c69ca79589bfd8";
```

定义订阅与发送地址

```java
    private final String pub_topic = "/sys/k28xr7bv6Qh/app_dev/thing/event/property/post";

    private final String sub_topic = "/sys/k28xr7bv6Qh/app_dev/thing/service/property/set";
```
云端连接方法
```java
private void mqtt_init() {
        try {

            String clientId = "a1MoTKOqkVK.test_device1";
            Map<String, String> params = new HashMap<String, String>(16);
            params.put("productKey", productKey);
            params.put("deviceName", deviceName);
            params.put("clientId", clientId);
            String timestamp = String.valueOf(System.currentTimeMillis());
            params.put("timestamp", timestamp);
            // cn-shanghai
            String host_url = "tcp://" + productKey + ".iot-as-mqtt.cn-shanghai.aliyuncs.com:1883";
            String client_id = clientId + "|securemode=2,signmethod=hmacsha1,timestamp=" + timestamp + "|";
            String user_name = deviceName + "&" + productKey;
            String password = com.example.dengzenmemiele_mqtt.AliyunloTSignUtil.sign(params, deviceSecret, "hmacsha1");

            //host为主机名，test为clientid即连接MQTT的客户端ID，一般以客户端唯一标识符表示，MemoryPersistence设置clientid的保存形式，默认为以内存保存
            System.out.println(">>>" + host_url);
            System.out.println(">>>" + client_id);

            //connectMqtt(targetServer, mqttclientId, mqttUsername, mqttPassword);

            client = new MqttClient(host_url, client_id, new MemoryPersistence());


            //MQTT的连接设置
            options = new MqttConnectOptions();
            //设置是否清空session,这里如果设置为false表示服务器会保留客户端的连接记录，这里设置为true表示每次连接到服务器都以新的身份连接
            options.setCleanSession(false);
            //设置连接的用户名
            options.setUserName(user_name);
            //设置连接的密码
            options.setPassword(password.toCharArray());
            // 设置超时时间 单位为秒
            options.setConnectionTimeout(10);
            // 设置会话心跳时间 单位为秒 服务器会每隔1.5*20秒的时间向客户端发送个消息判断客户端是否在线，但这个方法并没有重连的机制
            options.setKeepAliveInterval(60);
            //设置回调
            client.setCallback(new MqttCallback() {
                @Override
                public void connectionLost(Throwable cause) {
                    //连接丢失后，一般在这里面进行重连
                    System.out.println("connectionLost----------");
                }

                @Override
                public void deliveryComplete(IMqttDeliveryToken token) {
                    //publish后会执行到这里
                    System.out.println("deliveryComplete---------" + token.isComplete());
                }

                @Override
                public void messageArrived(String topicName, MqttMessage message)
                        throws Exception {
                    //subscribe后得到的消息会执行到这里面
                    System.out.println("messageArrived----------");
                    Message msg = new Message();
                    //封装message包
                    msg.what = 3;   //收到消息标志位
                    msg.obj = message.toString();
                    //发送messge到handler
                    handler.sendMessage(msg);    // hander 回传
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```



定义按钮功能

```java
        Button btn_open = findViewById(R.id.btn_open);
        Button btn_close = findViewById(R.id.btn_close);

        btn_open.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //开
                publish_message("{\"params\":{\"led\":1},\"version\":\"1.0.0\"}");
            }
        });
        btn_close.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //关
                publish_message("{\"params\":{\"led\":0},\"version\":\"1.0.0\"}");
            }
        });
```

引入mqtt初始化方法

```java
    @SuppressLint("RestrictedApi")
    private void mqtt_init() {
        try {

            String clientId = "a1MoTKOqkVK.test_device1";
            Map<String, String> params = new HashMap<String, String>(16);
            params.put("productKey", productKey);
            params.put("deviceName", deviceName);
            params.put("clientId", clientId);
            String timestamp = String.valueOf(System.currentTimeMillis());
            params.put("timestamp", timestamp);
            // cn-shanghai
            String host_url = "tcp://" + productKey + ".iot-as-mqtt.cn-shanghai.aliyuncs.com:1883";
            String client_id = clientId + "|securemode=2,signmethod=hmacsha1,timestamp=" + timestamp + "|";
            String user_name = deviceName + "&" + productKey;
            String password = com.example.dengzenmemiele_mqtt.AliyunloTSignUtil.sign(params, deviceSecret, "hmacsha1");

            //host为主机名，test为clientid即连接MQTT的客户端ID，一般以客户端唯一标识符表示，MemoryPersistence设置clientid的保存形式，默认为以内存保存
            System.out.println(">>>" + host_url);
            System.out.println(">>>" + client_id);

            //connectMqtt(targetServer, mqttclientId, mqttUsername, mqttPassword);

            client = new MqttClient(host_url, client_id, new MemoryPersistence());


            //MQTT的连接设置
            options = new MqttConnectOptions();
            //设置是否清空session,这里如果设置为false表示服务器会保留客户端的连接记录，这里设置为true表示每次连接到服务器都以新的身份连接
            options.setCleanSession(false);
            //设置连接的用户名
            options.setUserName(user_name);
            //设置连接的密码
            options.setPassword(password.toCharArray());
            // 设置超时时间 单位为秒
            options.setConnectionTimeout(10);
            // 设置会话心跳时间 单位为秒 服务器会每隔1.5*20秒的时间向客户端发送个消息判断客户端是否在线，但这个方法并没有重连的机制
            options.setKeepAliveInterval(60);
            //设置回调
            client.setCallback(new MqttCallback() {
                @Override
                public void connectionLost(Throwable cause) {
                    //连接丢失后，一般在这里面进行重连
                    System.out.println("connectionLost----------");
                }

                @Override
                public void deliveryComplete(IMqttDeliveryToken token) {
                    //publish后会执行到这里
                    System.out.println("deliveryComplete---------" + token.isComplete());
                }

                @Override
                public void messageArrived(String topicName, MqttMessage message)
                        throws Exception {
                    //subscribe后得到的消息会执行到这里面
                    System.out.println("messageArrived----------");
                    Message msg = new Message();
                    //封装message包
                    msg.what = 3;   //收到消息标志位
                    msg.obj = message.toString();
                    //发送messge到handler
                    handler.sendMessage(msg);    // hander 回传
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
mqtt链接
```java
    private void start_reconnect() {
        scheduler = Executors.newSingleThreadScheduledExecutor();
        scheduler.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                if (!client.isConnected()) {
                    mqtt_connect();
                }
            }
        }, 0 * 1000, 10 * 1000, TimeUnit.MILLISECONDS);
    }
```
mqtt订阅（回调）
```java
handler = new Handler() {
            @SuppressLint("SetTextI18n")
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                switch (msg.what) {
                    case 1: //开机校验更新回传
                        break;
                    case 2:  // 反馈回传
                        break;
                    case 3:  //MQTT 收到消息回传   UTF8Buffer msg=new UTF8Buffer(object.toString());
                        String message = msg.obj.toString();
                        Log.d("nicecode", "handleMessage: "+ message);
                        try {
                            JSONObject jsonObjectALL = null;
                            jsonObjectALL = new JSONObject(message);
                            JSONObject items = jsonObjectALL.getJSONObject("items");


                        } catch (JSONException e) {
                            e.printStackTrace();
                            break;
                        }
                        break;
                    case 30:  //连接失败
                        Toast.makeText(MainActivity.this, "连接失败", Toast.LENGTH_SHORT).show();
                        break;
                    case 31:   //连接成功
                        Toast.makeText(MainActivity.this, "连接成功", Toast.LENGTH_SHORT).show();
                        try {
                            client.subscribe(sub_topic, 1);
                        } catch (MqttException e) {
                            e.printStackTrace();
                        }
                        break;
                    default:
                        break;
                }
            }
        };
```

实现按钮功能（app数据向云端的发布）

```java
private void publish_message(String message) {
        if (client == null || !client.isConnected()) {
            return;
        }
        MqttMessage mqtt_message = new MqttMessage();
        mqtt_message.setPayload(message.getBytes());
        try {
            client.publish(pub_topic, mqtt_message);
        } catch (MqttException e) {
            e.printStackTrace();
        }
    }
```

修改app名称

![7518a470b8c11d377447579c853c18e](https://github.com/user-attachments/assets/332890fe-102e-4030-a167-9111df980239)


修改app图标

![2e67a20303f2e9fd269e960023d0f40](https://github.com/user-attachments/assets/26afc893-3361-4996-86f0-d7ee5c1e425d)

![0e0ab3b34b398768b16b9a1874fddec](https://github.com/user-attachments/assets/0a55708e-bcf1-4744-9186-c718972dea02)




### TASK 2.2.3
拓展3:使⽤阿⾥云物联⽹平台，从mcu往云端发送数据，并且在⼿机软件或web界⾯显⽰。（可以⾃⼰
设定⼀个数据，也可以使⽤mcu读取传感器数据）

这里我们可以读取esp32的ip

先修改arduino ide中的代码
只需加入这一行，先让单片机读取并上传自己的ip
```c++
    json_data["IP"] = WiFi.localIP().toString();
```
下面是软件的改进

先加入ip的显示
```java
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_marginTop="50dp"
        android:gravity="center">

        <TextView
            android:layout_width="103dp"
            android:layout_height="wrap_content"
            android:text="IP地址"
            android:textColor="@color/black"
            android:textSize="17dp" />

        <TextView
            android:id="@+id/tv_IP"
            android:layout_width="179dp"
            android:layout_height="wrap_content"
            android:text="0.0.0.0"
            android:textColor="@color/black"
            android:textSize="17dp" />

    </LinearLayout>
```
默认ip记为0.0.0.0
（顺便将上面两行也居中了）

![0195823a0cbb32d041e7421def02cca](https://github.com/user-attachments/assets/54323cfe-f430-43e6-b41a-eff721d29bd0)

编写显示控件
```java
        TextView tv_IP = findViewById(R.id.tv_IP);
```
在抓取函数中加入,并定义ipAddress
```java
                            JSONObject items = jsonObjectALL.getJSONObject("items");

                            JSONObject obj_IP = items.getJSONObject("IP");

                            ipAddress = obj_IP.getString("value");
```
成功实现

![3ec2bdb374b409ed935f134e9bf7f20](https://github.com/user-attachments/assets/0bd110c6-a53d-4cde-81c8-fbc74c0d9c9a)

与串口监视器中显示相同IP

![e187e53fae3a3ae77ec269b9a9d10c4](https://github.com/user-attachments/assets/1ce7265d-1428-4b49-9c3d-db1cd54192f4)


### TASK 2.2.4
拓展4:观察你⼿⾥⾯的ESP32-CAM，你会发现，他有⼀个摄像头。请将图像数据回传到云端，并且在
⾃⼰的⼿机软件或者web界⾯进⾏展⽰


#杂谈

出现灵异事件，烧录单片机后，他自己会受到off指令，也就是关灯指令，很灵异，之前是没有的

![6b69f148d0ff4cc9d11d65687eed76f](https://github.com/user-attachments/assets/c7a7c6f8-d3e7-4f8e-8303-fff2ab3bfb41)

发现是这个订阅链接在作祟

```
/sys/k28xr7bv6Qh/esp32cam/thing/event/property/post_reply
```
这个是单片机发送信息后，平台给你传送的连接成功的信息，可是被单片机读取了，以为是发出的控制指令

所以我们只需要忽略这条订阅链接发送的信息即可（我的链接可能跟你不一样）

```c++
  if (topic == "/sys/k28xr7bv6Qh/esp32cam/thing/event/property/post_reply") {
    Serial.println("Ignored message from topic: /sys/k28xr7bv6Qh/esp32cam/thing/event/property/post_reply");
    return; // 直接返回，不处理消息
  }
```
将这段代码补充道订阅链接函数 0nMqttMessage 中即可解决问题

![339af4ab432ef3eef8a135410a1e1bc](https://github.com/user-attachments/assets/4fd4b46b-f4ae-4b58-944e-99f5fcbfad4a)
