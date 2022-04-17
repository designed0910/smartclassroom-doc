# 温湿度传感器

## 供电信息

**DHT11的供电电压为3－5.5V。传感器上电后，要等待 1s 以越过不稳定状态在此期间无需发送任何指令。**

## 测量精度

|  型号   |      测量范围      | 测湿精度  | 测温精度 | 分辨力  |   封装   |
| :---: | :------------: | :---: | :--: | :--: | :----: |
| DHT11 | 20－90％RH 0－50℃ | ±5％RH | ±2℃  |  1   | 4针单排直插 |

## 传感器获取到的数据

|  温度   |  湿度   |
| :---: | :---: |
| 27.40 | 49.00 |



## 代码

```c
/* 库文件一般不用修改 */
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include "DHTesp.h"
#include <SimpleDHT.h>
/*
 * 我给出的代码的这部分是不用动的
 * 部分传感器需要多加一些库文件，例如温湿度传感器的库文件就很多，我加在下面，并注释掉，使用时只需取消注释即可（注意需要在IDE里面安装下面的库文件）
 * #include "DHTesp.h"
 * #include <SimpleDHT.h>
 * #include <FS.h>
 */

/* 这些不用动 */
#ifdef ESP32
#pragma message(THIS EXAMPLE IS FOR ESP8266 ONLY !)
#error Select ESP8266 board.
#endif

/* 自己定义的东西 ，例如下面的 LIGHTMODLE 5 就是把5这个引脚定义为 LIGHTMODLE，下同，可以自己在改动 */
#define SOILMODLE  5
#define SOUNDMODLE  16
int pinDHT11 = 2;

/* 连接火焰传感器 */


SimpleDHT11 dht11(pinDHT11);
DHTesp dht;

/* 下面是一些配置信息 */
const char  *ssid   = "Ustl";                /* WIFI名 */
const char  *password = "ustl@123";                /* 密码 */
const char  *mqtt_server  = "192.168.1.106";              /* mqtt服务器ip */
const char  *mqtt_user_name = "root";                       /* mqtt 用户名 */
const char  *mqtt_user_pswd = "root";                   /* mqtt密码 */


/* 下面这些东西要保证不重复，即在烧录进多个ESP8266时要保证下面信息的唯一性，可以自由改动 */
const char  *MODULE_ID  = "Byxj__001";           /* 本模块ID */
const char  *MODULE_PRO_ID  = "Byxj_8266_01";       /* 产品ID */
const char  *PROJECT_KEY  = "Byxj_Graduation";                        /* 项目的唯一ID */

/* 不用动，作用的WiFi模块声明（应该是） */
WiFiClient espClient;
PubSubClient client( espClient );


/* wifi初始化 */
void setup_wifi()
{
  delay( 10 );
  /* We start by connecting to a WiFi network */
  Serial.println();
  Serial.print( "Connecting to " );
  Serial.println( ssid );

  WiFi.begin( ssid, password );

  while ( WiFi.status() != WL_CONNECTED )
  {
    delay( 500 );
    Serial.print( "." );
  }
  randomSeed( micros() );
  Serial.println( "" );
  Serial.println( "WiFi connected" );
  Serial.println( "IP address: " );
  Serial.println( WiFi.localIP() );
}


/* 回调函数目前用不到，后期可以用于接收树莓派发送的消息 */
void callback( char *topic, byte *payload, unsigned int length )
{
  Serial.print( "Message arrived [" );
  Serial.print( topic );
  Serial.print( "] " );
  for ( int i = 0; i < length; i++ )
  {
    Serial.print( (char) payload[i] );
  }
  Serial.println();
}


/* 重连WiFi函数，不用动 */
void reconnect()
{
  /* Loop until we're reconnected */
  while ( !client.connected() )
  {
    Serial.print( "Attempting MQTT connection..." );
    if ( client.connect( MODULE_ID, mqtt_user_name, mqtt_user_pswd ) )
    {
      Serial.println( "connected" );
    }else  {
      Serial.print( "failed, rc=" );
      Serial.print( client.state() );
      Serial.println( " try again in 5 seconds" );
      /* Wait 5 seconds before retrying */
      delay( 5000 );
    }
  }
}


/* 不用动，具体功能就是初始化整个8266 */
void setup()
{
  Serial.begin( 9600 );
  String thisBoard = ARDUINO_BOARD;
  Serial.println( thisBoard );
  
  /*
   * Autodetect is not working reliable, don't use the following line
   * dht.setup(17);
   * use this instead:
   */

  /* 初始化 wifi */
  setup_wifi();

  client.setServer( mqtt_server, 1883 );
  client.connect( MODULE_ID, mqtt_user_name, mqtt_user_pswd );
  client.setCallback( callback );

  
}


/* folat char 转换，因为读取到的数据是float型，通过mqtt只能发送char型，故需转换 */
char *float2str( float val, int precision, char *buf )
{
  char *cur, *end;

  sprintf( buf, "%.6f", val );
  if ( precision < 6 )
  {
    cur = buf + strlen( buf ) - 1;
    end = cur - 6 + precision;
    while ( (cur > end) && (*cur == '0') )
    {
      *cur = '\0';
      cur--;
    }
  }

  return(buf);
}





/* 循环函数，初期所有改动均在这个函数里面 */
void loop()
{
  /* 下面用于重连，以免8266没网 */
  if ( !client.connected() )
  {
    reconnect();
  }
  client.loop();                          /*不用管，循环函数 */

  byte temperature = 0;
  byte humidity = 0;
  int err = SimpleDHTErrSuccess;
  if ((err = dht11.read(&temperature, &humidity, NULL)) != SimpleDHTErrSuccess) {
    Serial.print("Read DHT11 failed, err="); Serial.println(err); delay(1000);
    return;
  }

  Serial.print("Sample OK: ");
  Serial.print((int)temperature); Serial.print(" *C, ");
  Serial.print((int)humidity); Serial.println(" H");

  // DHT11 sampling rate is 1HZ.
   delay(1500);
  float sensorReading = analogRead(A0); 
  Serial.println(sensorReading);

  char firemsg[20];
  char hummsg[20];
  char tempemsg[20];
  char hummsg1[20] = "smart-hum-";
  char tempemsg1[20] = "smart-tem-";
  char firemsg1[20] = "smart-fire-";
  dtostrf(humidity, 3, 2, hummsg);
 
  dtostrf(temperature, 3, 2, tempemsg);
  strcat(hummsg1, hummsg);
  strcat(firemsg1,firemsg);

  strcat(tempemsg1, tempemsg);
  float2str(sensorReading, 1, firemsg);
  client.publish("smart-esp/hum", hummsg1);
  client.publish("smart-esp/tem", tempemsg1);
  client.publish("smart-esp/fire", firemsg1);
  
}

```

## 官方资料

[坚果云DH11](https://www.jianguoyun.com/p/DRbGVVYQ9NDYBhiY3ZwD)

