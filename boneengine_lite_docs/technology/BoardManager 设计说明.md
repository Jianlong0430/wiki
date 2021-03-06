## #概述
板级描述文件用来描述板级硬件资源和能力，板级配置文件是用户根据板级描述文件对外设或端口参数进行定制而生成的。

## #板级硬件资源

### #ESP32KIT
##### 开发板
![1.jpeg | left | 684x377](https://cdn.yuque.com/lark/2018/jpeg/66207/1523277514490-fd34eae8-d34d-44ab-8148-68ae95e448de.jpeg "")

##### 原理图
![1.png | left | 684x403](https://cdn.yuque.com/lark/2018/png/66207/1523276594348-8af3a920-a48a-4690-868f-f253cfe9af92.png "")


##### AliosThings支持情况
'Y'代表支持，'N'代表目前不支持。

| 功能/端口 | AliOSThings支持情况 |
| --- | --- |
| I2C | 'N' |
| ADC | 'N' |
| DAC | 'N' |
| GPIO | 'Y' |
| UART | 'Y' |
| PWM | 'Y' （仅支持pwm\_led） |
| WIFI | 'Y' |
| BT | 'Y' |
| WDG | 'N' |
| SD | 'N' |
| SENSOR | 'N' |
| SPI | 'Y' |
| I2S | 'N' |
| RF | 'N' |


##### 开发板IO/资源映射


| 功能/接口 | IO/资源映射 |
| --- | --- |
| I2C0 | SDA(IO2)  SCL(IO4) |
| I2C1 | SDA(IO5)  SCL(IO12) |
| ADC0 | ADC2\_CH4(IO13) |
| ADC1 | ADC2\_CH7(IO27) |
| UART0 | 默认板级log打印 |
| UART1 | TXD(IO17) RXD(IO16) CTS(IO15) RTS(IO14) |
| UART2 | TXD(IO32)  RXD(IO33) |
| GPIO | GPIO19(IO19) GPIO21(IO21) GPIO22(IO22) GPIO23(IO23) |
| DAC1 | DAC(IO25) |
| DAC2 | DAC(IO26) |
| PWM0 | PWM(IO34) |
| PWM1 | PWM(IO35) |
| WIFI | 内置 |
| BT | 内置 |
| WDG | 内置 |
| SD | SD0(SD0) SD1(SD1) SD2(SD2) SD3(SD3) CMD(CMD) CLK(CLK) |

备注：esp32Kit开发板把esp32的功能引脚全部引出了，用户可以自由定义各个管脚的功能，上面各个IO功能的分配参考了乐鑫 ESP-WROOM-32开发板的设计以及结合了Bone@Lite的具体应用场景，如果有需要，可以参考esp32的数据手册，对其进行调整。

## #板级描述文件

### #组织形式
板级描述文件采用json格式，其组织方式如下：
```json
{
    "模块0":[{子模块0},{子模块1}],
    "模块1":[{子模块0},{子模块1}],
}
```
其中"模块0"、"模块1"代表硬件的具体模块，比如GPIO模块，UART模块等等，"子模块0"、"子模块1"代表该模块中成员变量，比如GPIO模块中的"gpio\_0"和"gpio\_1"，UART模块中的"uart0"和"uart1"。

### #子模块对象
每个子模块都是一个json对象，模块中所有子模块都被组织到一个数组中。子模块的组织形式都是固定的，其基本模式如下：
```json
"id":子模块标志符,
"参数名":{
	"type":item成员类型,
	"fixed":item值是否可修改,
	"item":[value0,value1...],
    "range":[min,max],
},
```

* id:每个子模块都有一个id，类型是string，需要保证其全局唯一性；
* 参数名：每个模块自定义的参数名称，类型是string；
* type：描述item成员的数据类型，只支持两种类型：number和string;
* fixed:item是否可被修改，true表示可被修改，false表示不可修改;
* item:当fixed=true时，item数组表示该参数支持的选项列表；当fixed=false时，表示用户可以对item进行赋值和修改；
* range:当fixed=false时，range表示item的取值范围；

示范如下：
```json
#GPIO子模块示范
{
	"id":"gpio_0",
	"pin_config":{
	    "type":"string",
	    "fixed":true,
	    "item":["input_irq_rising","input_irq_down"],
		},
},

#UART子模块示范
{
	"id":"uart0",
	"data_width":{
		"type":"number",
		"fixed":true,
		"item":[5,6,7,8,9],
	},
	"baud_rate":{
		"type":"string",
		"fixed":true,
		"item":[9600,115200,921600],
	},
	"stop_bits":{
		"type":"number",
		"fixed":true,
		"item":[1,2],
	},
	"flow_control":{
		"type":"string",
		"fixed":true,
		"item":["disabled","cts","rts","cts_rts"],
	},
	"parity_config":{
		"type":"string",
		"fixed":true,
		"item":["no","odd","even"],
	},
},

```

## #板级配置文件

板级配置文件是根据板级描述文件生成的，去掉了type、fixed、range选项，比如：
```json
#GPIO子模块示范
{
	"id":"gpio_0",
	"pin_config":"input_irq_rising",
},

#UART子模块示范
{
	"id":"uart0",
	"data_width":8,
	"baud_rate":115200,
	"stop_bits":1,
	"flow_control":"disabled",
	"parity_config":"no",
},

#WIFI
WIFI:{
[
    "id":"wifi",
	"ssid":"test",
	"pwd":"hello",
],

},
```


## #完整示范
```json
#board description file
{
"GPIO":[
	{
		"id":"gpio_0",
		"pin_config":{
			"type":"string",
			"fixed":true,
			"item":["input_irq_rising","input_irq_down","input_irq_both","output_high","output_low"],
		},
	},
	{
		"id":"gpio_1",
		"pin_config":{
			"type":"string",
			"fixed":true,
			"item":["input_irq_rising","input_irq_down","input_irq_both","output_high","output_low"],
		},
	},
],


"UART":[

	{
		"id":"uart0",
		"data_width":{
			"type":"number",
			"fixed":true,
			"item":[5,6,7,8,9],
		},
		"baud_rate":{
			"type":"string",
			"fixed":number,
			"item":[9600,115200,921600],
		},
		"stop_bits":{
			"type":"number",
			"fixed":true,
			"item":[1,2],
		},
		"flow_control":{
			"type":"string",
			"fixed":true,
			"item":["disabled","cts","rts","cts_rts"],
		},
		"parity_config":{
			"type":"string",
			"fixed":true,
			"item":["no","odd","even"],
		},
	},
	{
		"id":"uart1",
		"data_width":{
			"type":"number",
			"fixed":true,
			"item":[5,6,7,8,9],
		},
		"baud_rate":{
			"type":"string",
			"fixed":number,
			"item":[9600,115200,921600],
		},
		"stop_bits":{
			"type":"number",
			"fixed":true,
			"item":[1,2],
		},
		"flow_control":{
			"type":"string",
			"fixed":true,
			"item":["disabled","cts","rts","cts_rts"],
		},
		"parity_config":{
			"type":"string",
			"fixed":true,
			"item":["no","odd","even"],
		},
	},
	
],
  

"I2C":[

	{
		"id":"i2c0",
		"i2c_freq":{
			"type":"number",
			"fixed":true,
			"item":[100,200,400],
		},
		"i2c_mode":{
			"type":"string",
			"fixed":true,
			"item":["slave","master","mem"],
		},
		"dev_addr":{
			"type":"number",
			"fixed":false,
			"item":[0x00],
			"range":[0,0xFFFF],
		},
	},
	{
		"id":"i2c1",
		"i2c_freq":{
			"type":"number",
			"fixed":true,
			"item":[100,200,400],
		},
		"i2c_mode":{
			"type":"string",
			"fixed":true,
			"item":["slave","master","mem"],
		},
		"dev_addr":{
			"type":"number",
			"fixed":false,
			"item":[0x00],
			"range":[0,0xFFFF],
		},
	},
	
],


"WIFI":[
	{
        "id":"wifi",
		"ssid":{
			"type":"string",
			"fixed":false,
			"item":["test"],
			"range":[0,64],
		},
		"pwd":{
			"type":"string",
			"fixed":false,
			"item":["test"],
			"range":[0,64],
		},
	}
],

}
```
对应的板级配置文件：
```json
#board config file

{
“GPIO”:[
	{
		"id":"gpio_0",
		"pin_config":”input_irq_rising“,
	},
	{
		"id":"gpio_55",
		"pin_config":output_high,
	},
],

“UART”:[

	{
		"id":"uart0",
		"data_width":8,
		"baud_rate":115200,
		"stop_bits":1,
		"flow_control":"disabled",
		"parity_config":"no",
	},
	{
		"id":"uart1",
		"data_width":8,
		"baud_rate":9600,
		"stop_bits":1,
		"flow_control":"disabled",
		"parity_config":"no",
	},
],
  
“I2C”:[

	{
		"id":"i2c0",
		"i2c_freq":200,
		"i2c_mode":"slave",
		"dev_addr":0x55,
	},
	{
		"id":"i2c1",
		"i2c_freq":100,
		"i2c_mode":"master",
		"dev_addr":0x12,
	},
],

“WIFI”:[
	{
       "id":"wifi",
		"ssid":"hello",
		"pwd":"1234567",
	}
],

}
```


## #参考
* 《ESP32\_DEVKITC\_V4-SCH-20171206A.pdf》
* 《esp32\_hardware\_design\_guidelines\_cn.pdf》
* 《esp32\_technical\_reference\_manual\_cn.pdf》
* 《esp32\_datasheet\_cn.pdf》
* [https://github.com/alibaba/AliOS-Things](https://github.com/alibaba/AliOS-Things)

 