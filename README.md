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
