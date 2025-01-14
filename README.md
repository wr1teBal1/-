# 许恒睿——嵌入式赛道仓库

## TASK 1
环境配置成功，编译并烧录成功如下图
![4c60c297f75203a9a72f95b8b0b5a9b](https://github.com/user-attachments/assets/7bc869df-b99c-4425-bcfc-c58a689aac33)

烧录效果图

https://github.com/user-attachments/assets/d8208729-484a-4431-81bb-733bfd2f66a7

源码如下，运用示例中的basics，重新定义esp32 cam的led引脚pin 4即可

```arduino
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
### TASK 2.1
学习esp32单⽚机的WiFi模块，连接⾃⼰的⼿机热点
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
