# 许恒睿——嵌入式赛道仓库

## TASK 1
环境配置成功，编译并烧录成功如下图
![4c60c297f75203a9a72f95b8b0b5a9b](https://github.com/user-attachments/assets/7bc869df-b99c-4425-bcfc-c58a689aac33)

烧录效果图

https://github.com/user-attachments/assets/d8208729-484a-4431-81bb-733bfd2f66a7

源码如下，运用示例中的basics，重新定义esp32 cam的led引脚pin 4即可

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
