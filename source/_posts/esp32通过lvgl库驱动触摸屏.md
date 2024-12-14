---
title: esp32通过lvgl库驱动触摸屏
date: 2023-08-06 23:11:32
tags: ESP32
---

在ESP32开发板上使用LVGL框架开发LCD显示屏界面

显示框架：LVGL

TFT芯片：ST7789

电容触摸芯片：CST816S

![run_demo](run_demo.png)

## 1.TFT_eSPI drive LCD

https://github.com/Bodmer/TFT_eSPI/discussions/2555

This wiki page was useful: [Github](https://github.com/Bodmer/TFT_eSPI/wiki/Installing-on-PlatformIO)

That said, there are some missing parts that due to the state of the library will show up as a non-functioning screen. For my use case (an ILI9488 device), I needed to provide additional values. I believe some of these are necessary for all projects.

Here is the bare minimum project using Visual Studio Code with PlatformIo and using the technique of platformio.ini file modification. No library file modification needed!

platformio.ini:

```
[env:nodemcu-32s]
platform = espressif32
board = nodemcu-32s
framework = arduino
monitor_speed = 115200
lib_deps = bodmer/TFT_eSPI@^2.5.23
build_flags =
-D USER_SETUP_LOADED
-D ILI9488_DRIVER
-D TFT_MISO=19
-D TFT_MOSI=23
-D TFT_SCLK=18
-D TFT_CS=15
-D TFT_DC=2
-D TFT_RST=4
-D LOAD_GLCD=1
-D SMOOTH_FONT
-D SPI_FREQUENCY=27000000
```

main.cpp:

```c++
#include <Arduino.h>
#include <TFT_eSPI.h> // Hardware-specific library
TFT_eSPI tft = TFT_eSPI(); // Invoke custom library

void setup()
{
Serial.begin(115200); // For debug
tft.init();

tft.setRotation(1);
tft.fillScreen(TFT_GREEN);
}

void loop()
{
// put your main code here, to run repeatedly:
}
```

Note that I had to include USER_SETUP_LOADED as well as SMOOTH_FONT.

There are some minor errors in the wiki article. It projects like we need to have an =1 but I found it works just like the include system and is not necessary.


## 2.My project, need offset.

platformio.ini:

```c++
[env:wesp32]
platform = espressif32
board = wesp32
framework = arduino
monitor_speed = 115200
upload_speed = 230400
board_build.partitions = default_16MB.csv
board_upload.flash_size = 16MB
lib_deps = 
	bodmer/TFT_eSPI@^2.5.30
    lvgl/lvgl@^8.3.7
build_flags = 
	-D USER_SETUP_LOADED
	-D ST7789_DRIVER
	-D CGRAM_OFFSET
	-D TFT_MOSI=23
	-D TFT_SCLK=18
	-D TFT_DC=2
	-D TFT_RST=4
	-D LOAD_GLCD=1
	-D SMOOTH_FONT
	-D SPI_FREQUENCY=55000000
```

## 3.lvgl

main.cpp
```c++
/*Using LVGL with Arduino requires some extra steps:
 *Be sure to read the docs here: https://docs.lvgl.io/master/get-started/platforms/arduino.html  */

#include <lvgl.h>
#include <TFT_eSPI.h>

/*To use the built-in examples and demos of LVGL uncomment the includes below respectively.
 *You also need to copy `lvgl/examples` to `lvgl/src/examples`. Similarly for the demos `lvgl/demos` to `lvgl/src/demos`.
 Note that the `lv_examples` library is for LVGL v7 and you shouldn't install it for this version (since LVGL v8)
 as the examples and demos are now part of the main LVGL library. */

/*Change to your screen resolution*/
static const uint16_t screenWidth  = 240;
static const uint16_t screenHeight = 280;

static lv_disp_draw_buf_t draw_buf;
static lv_color_t buf[ screenWidth * 10 ];

TFT_eSPI tft = TFT_eSPI(screenWidth, screenHeight); /* TFT instance */

#if LV_USE_LOG != 0
/* Serial debugging */
void my_print(const char * buf)
{
    Serial.printf(buf);
    Serial.flush();
}
#endif

/* Display flushing */
void my_disp_flush( lv_disp_drv_t *disp, const lv_area_t *area, lv_color_t *color_p )
{
    uint32_t w = ( area->x2 - area->x1 + 1 );
    uint32_t h = ( area->y2 - area->y1 + 1 );

    tft.startWrite();
    tft.setAddrWindow( area->x1, area->y1, w, h );
    tft.pushColors( ( uint16_t * )&color_p->full, w * h, true );
    tft.endWrite();

    lv_disp_flush_ready( disp );
}

void setup()
{
    Serial.begin( 115200 ); /* prepare for possible serial debug */

    String LVGL_Arduino = "Hello Arduino! ";
    LVGL_Arduino += String('V') + lv_version_major() + "." + lv_version_minor() + "." + lv_version_patch();

    Serial.println( LVGL_Arduino );
    Serial.println( "I am LVGL_Arduino" );

    lv_init();

#if LV_USE_LOG != 0
    lv_log_register_print_cb( my_print ); /* register print function for debugging */
#endif

    tft.begin();          /* TFT init */
    tft.setRotation( 4 ); /* 2:top fpc 4:bottom fpc */

    lv_disp_draw_buf_init( &draw_buf, buf, NULL, screenWidth * 10 );

    /*Initialize the display*/
    static lv_disp_drv_t disp_drv;
    lv_disp_drv_init( &disp_drv );
    /*Change the following line to your display resolution*/
    disp_drv.hor_res = screenWidth;
    disp_drv.ver_res = screenHeight;
    disp_drv.flush_cb = my_disp_flush;
    disp_drv.draw_buf = &draw_buf;
    lv_disp_drv_register( &disp_drv );

    /* Create simple label */
    lv_obj_t *label_top = lv_label_create( lv_scr_act() );
    lv_label_set_text( label_top, LVGL_Arduino.c_str() );
    lv_obj_align( label_top, LV_ALIGN_TOP_LEFT, 0, 0 );

    lv_obj_t *label = lv_label_create( lv_scr_act() );
    lv_label_set_text( label, LVGL_Arduino.c_str() );
    lv_obj_align( label, LV_ALIGN_CENTER, 0, 0 );

    lv_obj_t *label_bottom = lv_label_create( lv_scr_act() );
    lv_label_set_text( label_bottom, LVGL_Arduino.c_str() );
    lv_obj_align( label_bottom, LV_ALIGN_BOTTOM_RIGHT, 0, 0 );

    Serial.println( "Setup done" );
}

void loop()
{
    lv_timer_handler(); /* let the GUI do its work */
    delay( 5 );
}
```

## 4.lvgl+touch
platformio.ini:
```c++
[env:wesp32]
platform = espressif32
board = wesp32
framework = arduino
monitor_speed = 115200
upload_speed = 230400
board_build.partitions = default_16MB.csv
board_upload.flash_size = 16MB
lib_deps = 
	bodmer/TFT_eSPI@^2.5.30
    fbiego/CST816S@^1.1.0
    lvgl/lvgl@^8.3.7
build_flags = 
	-D USER_SETUP_LOADED
	-D ST7789_DRIVER
	-D CGRAM_OFFSET
	-D TFT_MOSI=23
	-D TFT_SCLK=18
	-D TFT_DC=2
	-D TFT_RST=4
	-D LOAD_GLCD=1
	-D SMOOTH_FONT
	-D SPI_FREQUENCY=55000000
```

main.cpp
```c++
/*Using LVGL with Arduino requires some extra steps:
 *Be sure to read the docs here: https://docs.lvgl.io/master/get-started/platforms/arduino.html  */

#include <lvgl.h>
#include <TFT_eSPI.h>
#include <CST816S.h>


/*To use the built-in examples and demos of LVGL uncomment the includes below respectively.
 *You also need to copy `lvgl/examples` to `lvgl/src/examples`. Similarly for the demos `lvgl/demos` to `lvgl/src/demos`.
 Note that the `lv_examples` library is for LVGL v7 and you shouldn't install it for this version (since LVGL v8)
 as the examples and demos are now part of the main LVGL library. */

/*Change to your screen resolution*/
static const uint16_t screenWidth  = 240;
static const uint16_t screenHeight = 280;

static lv_disp_draw_buf_t draw_buf;
static lv_color_t buf[ screenWidth * 10 ];

TFT_eSPI tft = TFT_eSPI(screenWidth, screenHeight); /* TFT instance */

CST816S touch(15, 4, 5, 34);	// sda, scl, rst, irq

#if LV_USE_LOG != 0
/* Serial debugging */
void my_print(const char * buf)
{
    Serial.printf(buf);
    Serial.flush();
}
#endif

/* Display flushing */
void my_disp_flush( lv_disp_drv_t *disp, const lv_area_t *area, lv_color_t *color_p )
{
    uint32_t w = ( area->x2 - area->x1 + 1 );
    uint32_t h = ( area->y2 - area->y1 + 1 );

    tft.startWrite();
    tft.setAddrWindow( area->x1, area->y1, w, h );
    tft.pushColors( ( uint16_t * )&color_p->full, w * h, true );
    tft.endWrite();

    lv_disp_flush_ready( disp );
}

/*Read the touchpad*/
void my_touchpad_read(lv_indev_drv_t * indev_driver, lv_indev_data_t * data )
{
    bool touched = touch.available();

    if( !touched )
    {
        data->state = LV_INDEV_STATE_REL;
    }
    else
    {
        data->state = LV_INDEV_STATE_PR;

        /*Set the coordinates*/
        data->point.x = touch.data.x;
        data->point.y = touch.data.y;

        Serial.print( "Data x " );
        Serial.println( touch.data.x );

        Serial.print( "Data y " );
        Serial.println( touch.data.y );
    }
}

void setup()
{
    Serial.begin( 115200 ); /* prepare for possible serial debug */

    String LVGL_Arduino = "Hello Arduino! ";
    LVGL_Arduino += String('V') + lv_version_major() + "." + lv_version_minor() + "." + lv_version_patch();

    Serial.println( LVGL_Arduino );
    Serial.println( "I am LVGL_Arduino" );

    lv_init();

#if LV_USE_LOG != 0
    lv_log_register_print_cb( my_print ); /* register print function for debugging */
#endif

    tft.begin();          /* TFT init */
    tft.setRotation( 2 ); /* 2:top fpc 4:bottom fpc */

    touch.begin();
    Serial.print(touch.data.version);
    Serial.print("\t");
    Serial.print(touch.data.versionInfo[0]);
    Serial.print("-");
    Serial.print(touch.data.versionInfo[1]);
    Serial.print("-");
    Serial.println(touch.data.versionInfo[2]);

    lv_disp_draw_buf_init( &draw_buf, buf, NULL, screenWidth * 10 );

    /*Initialize the display*/
    static lv_disp_drv_t disp_drv;
    lv_disp_drv_init( &disp_drv );
    /*Change the following line to your display resolution*/
    disp_drv.hor_res = screenWidth;
    disp_drv.ver_res = screenHeight;
    disp_drv.flush_cb = my_disp_flush;
    disp_drv.draw_buf = &draw_buf;
    lv_disp_drv_register( &disp_drv );

    /*Initialize the (dummy) input device driver*/
    static lv_indev_drv_t indev_drv;
    lv_indev_drv_init( &indev_drv );
    indev_drv.type = LV_INDEV_TYPE_POINTER;
    indev_drv.read_cb = my_touchpad_read;
    lv_indev_drv_register( &indev_drv );

    /* Create simple label */
    lv_obj_t *label_top = lv_label_create( lv_scr_act() );
    lv_label_set_text( label_top, LVGL_Arduino.c_str() );
    lv_obj_align( label_top, LV_ALIGN_TOP_LEFT, 0, 0 );

    lv_obj_t *label = lv_label_create( lv_scr_act() );
    lv_label_set_text( label, LVGL_Arduino.c_str() );
    lv_obj_align( label, LV_ALIGN_CENTER, 0, 0 );

    lv_obj_t *label_bottom = lv_label_create( lv_scr_act() );
    lv_label_set_text( label_bottom, LVGL_Arduino.c_str() );
    lv_obj_align( label_bottom, LV_ALIGN_BOTTOM_RIGHT, 0, 0 );

    Serial.println( "Setup done" );
}

void loop()
{
    lv_timer_handler(); /* let the GUI do its work */
    delay( 5 );
}

```

## 5.lvgl demo

使用官方demos流程：

* lv_conf.h 中 `#define LV_USE_DEMO_WIDGETS 1`
* 将lvgl库中的demos移动到src文件夹。不然会出现`undefined reference to 'lv_demo_benchmark' `（例如lv_demo_benchmark） 这是由于lv_demo_benchmark.c未被编译导致的 把demos文件夹移动到lvgl的src目录下就行了，这样就会参与到编译中了
* 包含头文件，例如在main.c文件中 `#include "demos/lv_demos.h"`
* 初始化等参考官方例程，setup中调用demo中的函数即可。 `lv_demo_widgets();`


## 面包板测试

注意核对FPC座的线序(1脚的位置)

| ESP32 | FPC | Function | Other             |
| ----- | --- | -------- | ----------------- |
| GND   | 1   | GND      |                   |
| GND   | 2   | LED_K    | 可通过PWM调整亮度 |
| 3V3   | 3   | VDD      | 供电              |
| 3V3   | 4   | VDD      |                   |
| GND   | 5   | GND      |                   |
| GND   | 6   | GND      |                   |
| 2     | 7   | D/C      |                   |
| GND   | 8   | CS       | 默认拉低一直有效  |
| 18    | 9   | SCL      | SPI_SCL           |
| 23    | 10  | SDA      | SPI_MOSI          |
| 32    | 11  | REST     | LCD_RST           |
| GND   | 12  | GND      |                   |
| 4     | 13  | TP_SCL   | I2C0_SCL          |
| 15    | 14  | TP_SDA   | I2C0_SDA          |
| 5     | 15  | TP_RST   | Touch_RST         |
| 34    | 16  | TP_INT   | Touch_INT         |
| 3V3   | 17  | VDD      |                   |
| GND   | 18  | GND      |                   |

## 6.使用自己设计的PCB板

Controler_ESP32_V0.1（PFC座方向反了）

| ESP32        | FPC | Function | Other               |
| ------------ | --- | -------- | ------------------- |
| GND          | 1   | GND      |                     |
| GND          | 2   | LED_K    | 可通过PWM调整亮度   |
| 3V3          | 3   | VDD      | 供电                |
| 3V3          | 4   | VDD      |                     |
| GND          | 5   | GND      |                     |
| GND          | 6   | GND      |                     |
| 2            | 7   | D/C      |                     |
| 32           | 8   | CS       |                     |
| 18           | 9   | SCL      | SPI_SCL             |
| 23           | 10  | SDA      | SPI_MOSI            |
| PCA9535 IO10 | 11  | REST     | IO扩展芯片 I2C0控制 |
| GND          | 12  | GND      |                     |
| 4            | 13  | TP_SCL   | I2C0_SCL            |
| 15           | 14  | TP_SDA   | I2C0_SDA            |
| PCA9535 IO11 | 15  | TP_RST   | IO扩展芯片 I2C0控制 |
| 34           | 16  | TP_INT   | Touch_INT           |
| 3V3          | 17  | VDD      |                     |
| GND          | 18  | GND      |                     |



