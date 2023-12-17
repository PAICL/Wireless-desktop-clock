# Wireless-desktop-clock
Wireless desktop clock.

There is nothing here :(
I'm still learning how to publish a good project.
Forgive me for this paper may be a bit oversimplified.
The demoboard is Seeed XIAO ESP32-S3.
programming in Ardino IDE.
It may require you to add some libraries.


---
~~~C
//库
#include <Wire.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <NTPClient.h>
#include <WiFiUdp.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <time.h>

//显示模块设置
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 32
#define OLED_RESET     -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

//WiFi名称与密码设置
const char* ssid = "Here is your WiFi's name";
const char* password = "Here is your WiFi's password";

//实时天气API接口调用及设置
String apiKey = "Here is a API, and you need to seiting you weather API";
String city = "Here is you local city";
String country = "Here is your country";
String url = "Here is you weather API website" + city + "," + country + "&appid=" + apiKey;

//实时时间调用
WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "pool.ntp.org");

//设置静态变量参数：温度（开尔文）、天气
static double temp_K = 0;
static String aweather;

//光遇图像
static const unsigned char PROGMEM myBitmap[] = {
0x00,0x00,0x00,0x00,0x00,0x03,0x00,0x00,0x00,0x06,0x00,0x00,0x00,0x06,0x00,0x00,0x00,0x0E,0xF8,0x00,0x00,0x31,0x0C,0x00,0x00,0x60,0x02,0x00,0x00,0x40,0x03,0x00,
0x00,0x81,0x01,0x00,0x00,0x81,0x80,0x80,0x01,0x03,0xD0,0x80,0x03,0x07,0xF8,0xC0,0x02,0x0F,0xFA,0x60,0x04,0x2F,0xFA,0x34,0x38,0x2F,0xFA,0x18,0x1C,0x2F,0xFE,0xE0,
0x03,0xB7,0xE7,0x80,0x01,0x58,0x1A,0x00,0x00,0x37,0xEA,0x00,0x04,0x48,0x0A,0x20,0x0A,0x4F,0xF2,0x50,0x04,0x31,0x86,0x20,0x00,0x20,0x82,0x00,0x00,0x40,0x81,0x00,
0x00,0x80,0x81,0x80,0x01,0x80,0x80,0xC0,0x01,0xA0,0x81,0xC0,0x00,0x5A,0x17,0x00,0x00,0x05,0xE8,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
};

static const unsigned char PROGMEM image[] ={
0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x7F,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x7F,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x7F,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFE,0x3F,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xEF,0xFF,0xFF,0xFF,0xFE,0x3F,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xC7,0xFF,0xFF,0xEF,0xFC,0x1F,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0x83,0xFF,0xFF,0xC7,0xF8,0x0F,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x00,0x00,0x00,0x03,0xFC,0x1F,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0x83,0xFD,0xFF,0xC7,0xFE,0x3F,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xC7,0xF8,0xFF,0xEC,0xCF,0x79,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xC6,0x7D,0xCC,0x00,0x40,0x00,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xC6,0x00,0x07,0xC0,0x47,0x70,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xC6,0x3F,0xC4,0x40,0xC6,0x30,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xC6,0x00,0x44,0x47,0xC4,0x11,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xC6,0x3F,0xC4,0x44,0x40,0x01,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xC6,0x3F,0xC4,0x44,0x44,0x11,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0x86,0x00,0x04,0x44,0x46,0x31,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x86,0x78,0xCC,0x44,0x47,0x71,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0x8E,0xD8,0xFC,0xC4,0x40,0x01,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x0F,0x98,0xC8,0xC4,0x47,0x79,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0x1E,0x08,0x81,0xC4,0xCE,0x3D,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x1C,0x38,0xE3,0xC7,0xFC,0x1F,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFE,0x7E,0x78,0xCF,0xC7,0xF8,0x0F,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFC,0xFF,0x39,0x9F,0xCF,0xFC,0x1F,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFB,0xFF,0xDF,0xFE,0x3F,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFE,0x3F,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x7F,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x7F,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF};/*"C:\Users\22945\Desktop\yuanshengqidong.BMP",0*/


void setup() {
  //端口设置
  Serial.begin(115200);
  delay(1000);
  
  //初始化屏幕
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("SSD1306 allocation failed"));
    for(;;);
  }

  display.clearDisplay();
  display.drawBitmap(0, 0, image, 128, 32, WHITE);
  display.display();
  delay(7000);
  display.clearDisplay();
  display.display();

  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(WHITE);
  display.setCursor(0,0);
  display.println("Connecting to WiFi...");
  display.display();
  display.clearDisplay();

  //WiFi检测
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting...");
  }
  display.clearDisplay();
  display.setCursor(0,0);
  display.println("WiFi connected");
  display.display();
  display.clearDisplay();

  // 设置时区，例如东八区为3600*8
  timeClient.begin();
  timeClient.setTimeOffset(3600*8);

  //获取日期：年/月/日
  //初始化NTP
  configTime(0, 0, "pool.ntp.org");  // 使用NTP服务器获取时间
  delay(1000);
  while (!time(nullptr)) {  // 等待直到获取到时间
    Serial.println("Waiting for time...");
    delay(1000);
  }

  //调用一次天气状况，为下面天气显示做第一次显示
  if ((WiFi.status() == WL_CONNECTED)) {
    HTTPClient http;
    http.begin(url);
    int httpCode = http.GET();
    if (httpCode > 0) {
      String payload = http.getString();
      DynamicJsonDocument doc(1024);
      deserializeJson(doc, payload);
      aweather = (doc["weather"][0]["main"]).as<String>();
      temp_K = doc["main"]["temp"];
    }
  }

}//void setup()结束

//循环开始
void loop() {
  //设置静态变量参数时间，用于为天气每60秒读取一次天气状况
  static unsigned long lastUpdateTime = 0;

  //没300秒通过调用天气API读取天气状况
  if ((WiFi.status() == WL_CONNECTED) && (millis() - lastUpdateTime >= 300000)) {
    lastUpdateTime = millis();
    HTTPClient http;
    http.begin(url);
    int httpCode = http.GET();
    if (httpCode > 0) {
      String payload = http.getString();
      DynamicJsonDocument doc(1024);
      deserializeJson(doc, payload);
      aweather = (doc["weather"][0]["main"]).as<String>();
      temp_K = doc["main"]["temp"];
    }
  }

  //将读取到的开尔文温度转换为摄氏度
  double temp_C = temp_K - 273.15;
  
  //更新时间
  timeClient.update();

  //跟新日期信息
  time_t now = time(nullptr);  // 获取当前时间
  struct tm* timeinfo = localtime(&now);  // 将时间转换为tm结构体

  int year = timeinfo->tm_year + 1900;  // 年份
  int month = timeinfo->tm_mon + 1;  // 月份
  int day = timeinfo->tm_mday - 1;  // 日期
  int dayOfWeek = timeinfo->tm_wday;  // 星期，0表示星期日，1-6表示星期一至星期六
  //int dayOfWeek = timeClient.getDay();

  //dayOfWeek是数字的周天数
  String weekDays[7] = {"Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"};
  String dayOfWeekStr = weekDays[dayOfWeek];

  //dayOfMouth是数字的月天数
  String monthDays[12] = {"Jan", "Feb", "Mar", "Apr", "may", "Jun", "Jul", "Aug", "Sept", "Oct", "Nov", "Dec"};
  String dayOfMonthStr = monthDays[month-1];

  //清屏
  display.clearDisplay();

  //设置显示位置
  display.setCursor(0,0);
  //显示日期或者天气状况
  //设置切换时间
  static unsigned long lastSwitchDateTime = 0;
  static bool showWeather = true;
  if (millis() - lastSwitchDateTime >= 3000) {
    lastSwitchDateTime = millis();
    showWeather = !showWeather;
  }
  if (showWeather) {
    display.println(aweather + (" ") +String(temp_C) + "'C");
  } else  {
   display.setCursor(0,0);
    display.println(dayOfMonthStr + (" ") + String(day+1) + (" ") + dayOfWeekStr);  // 显示日期
  }

  //设置显示位置
  display.setCursor(0,8);
  //在屏幕上显示实时时间和标语内容
  display.setTextSize(2);
  display.println(timeClient.getFormattedTime());
  display.setTextSize(1);

  //设置一个判断语句用来在不同时间显示不同文本
  String formattedTime = timeClient.getFormattedTime();//设置一个字符串读取时间
  int hour = formattedTime.substring(0, 2).toInt();//设置时间的小时为时间的小时的两位数和中间的冒号（时间分隔符）
  //int minute = formattedTime.substring(3, 5).toInt();//设置时间的分钟为时间的小时的两位数和中间的冒号（时间分隔符）
  if (hour >= 23 || hour < 6){
    display.println("Time To Sleep.");
  } else if (hour >= 6 && hour <23 ) {
    display.println("Full Of Energy!");
  }

  static unsigned long lastUpdateTime_anisky = 0;

  //显示光遇图像
  display.drawBitmap(96, 0, myBitmap, 32, 32, WHITE);
  display.display();
  display.clearDisplay();

}//循环结束语句

~~~
