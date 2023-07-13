# ZS-SHT30-1T-1H-485 通讯协议解码器

## 介绍
``` 
制造商：中盛科技
产品名称：温湿度采集模块
产品型号：ZS-SHT30-1T-1H-485
官网连接：http://www.zskjdg.com/Product/productChild?childId=2&ProductType=humiture
``` 

## 资料
[协议文档](http://www.zskjdg.com/dev-api/common/download?fileName=%E4%BA%A7%E5%93%81%E8%B5%84%E6%96%99/6%E6%B8%A9%E5%BA%A6%E9%87%87%E9%9B%86%E7%B3%BB%E5%88%97/RS485/%E6%B8%A9%E6%B9%BF%E5%BA%A6%E9%87%87%E9%9B%86%E7%B3%BB%E5%88%97%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C%EF%BC%88RS485%E7%89%88%EF%BC%89.pdf)

![image](http://docs.fox-tech.cn/_images/ZS-SHT30-1T-1H-485-01.png)
## 工具

``` 
推荐工具：
MODBUS调试助手
``` 

## 组网

![image](http://docs.fox-tech.cn/_images/ZS-SHT30-1T-1H-485.jpg)

## 源码

- fox-edge-server-protocol-zs-sht30-1t-1h

[源码](https://gitee.com/fierce_wolf/fox-edge-server/tree/master/fox-edge-server-protocol/fox-edge-server-protocol-zs-sht30-1t-1h)

### 关键JAVA代码

- 为设备的一个操作创建一个JAVA类

可为设备的每一个操作接口，建立一个JAVA类，代码管理起来比较简洁。<br>
在java类上加上注解FoxEdgeDeviceType，告诉Fox-Edge这个哪家厂商的哪款设备。Fox-Edge的Device服务在启动的时候，会扫描解码器jar包的信息。
此时，它可以通过这个组件，获知可管理的设备的信息。<br>
比如，这边定义了中盛科技的温湿度采集模块(ZS-SHT30-1T-1H-485)的设备。<br>

```js
@FoxEdgeDeviceType(value = "温湿度采集模块(ZS-SHT30-1T-1H-485)", manufacturer = "中盛科技")
public class ZSKJReadTemperatureAndHumidity {
 ....
 ..
 .
}

```

- 为设备的一个操作添加一堆编码/解码函数

需要为设备操作，添加一对static的编码/解码函数，用于通信数据的编码和解码。<br>
在编码函数/解码函数上加上注解FoxEdgeOperate，告诉Fox-Edge对应的操作名称和这是编码/解码函数，建议的通信超时是多长。
Fox-Edge的Device服务在启动的时候，会扫描解码器jar包的信息。此时，它可以通过这个组件，获知可管理的设备的操作信息。<br>
比如，这边定义了读取温湿度的一对编码/解码函数<br>


```js
/**
 * 读取Registers
 */
@FoxEdgeDeviceType(value = "温湿度采集模块(ZS-SHT30-1T-1H-485)", manufacturer = "中盛科技")
public class ZSKJReadTemperatureAndHumidity {
    /**
     * 编码函数
     *
     * @param param 编码参数，它来自设备配置的参数和操作任务配置的参数的合并
	 *
     * @return 编码结果 该数据的对象类型，要根据你准备转发的Channel服务来定义。
	 * 比如，serial-channel提供的数据格式，是16进制格式的来表示通信报文，所以你要返回一个hex通信报文格式的String给外部
     */
    @FoxEdgeOperate(name = "读取温湿度", polling = true, type = FoxEdgeOperate.encoder, timeout = 2000)
    public static String packReadInputRegister(Map<String, Object> param) {
        return (String) operateReadRegister("", param);
    }

    /**
     * 解码函数
     *
     * @param hexString 该对象是由你使用的channnel从设备获得的报文，以hex格式文本输入给你。
	 * 如果你用的是其他channel，其他channel有自己的约定，那么你需要采用对应的格式。
	 * 目前各类channel支持的有常见的hexString，也有结构复杂一点的Map<String, Object>
	 *
     * @param param  解码参数，它来自设备配置的参数和操作任务配置的参数的合并，到时候你知道该如何对设备返回报文进行解码
     * @return 解码后的数据 必须为Map<String, Object>格式，表示一批key-value的设备数据对象和它的数值
     */
    @FoxEdgeOperate(name = "读取温湿度", polling = true, type = FoxEdgeOperate.decoder, timeout = 2000)
    public static Map<String, Object> unpackReadInputRegister(String hexString, Map<String, Object> param) {
        return (Map<String, Object>) operateReadRegister(hexString, param);
    }
}

```

### 完整JAVA代码

```js
package cn.foxtech.device.protocol.zs.sht301t1h;

import cn.foxtech.device.protocol.core.annotation.FoxEdgeDeviceType;
import cn.foxtech.device.protocol.core.annotation.FoxEdgeOperate;
import cn.foxtech.device.protocol.core.exception.ProtocolException;
import cn.foxtech.device.protocol.core.utils.HexUtils;
import cn.foxtech.device.protocol.core.utils.MethodUtils;
import cn.foxtech.device.protocol.modbus.core.ModBusConstants;
import cn.foxtech.device.protocol.modbus.core.ModBusProtocol;
import cn.foxtech.device.protocol.modbus.core.ModBusProtocolFactory;
import cn.foxtech.device.protocol.modbus.core.ModBusReadRegistersRespond;

import java.util.HashMap;
import java.util.Map;

/**
 * 读取Registers
 */
@FoxEdgeDeviceType(value = "温湿度采集模块(ZS-SHT30-1T-1H-485)", manufacturer = "中盛科技")
public class ZSKJReadTemperatureAndHumidity {
    /**
     * 读取保持寄存器
     *
     * @param param 必须包含device_addr和modbus_holding_registers_template两个输入参数
     * @return 16进制文本格式的报文
     */
    @FoxEdgeOperate(name = "读取温湿度", polling = true, type = FoxEdgeOperate.encoder, timeout = 2000)
    public static String packReadInputRegister(Map<String, Object> param) {
        return (String) operateReadRegister("", param);
    }

    /**
     * 解码保持寄存器
     *
     * @param hexString 16进制文本格式的报文
     * @param param     必须包含 device_addr 和 modbus_holding_registers_template 两个输入参数
     * @return 解码后的数据
     */
    @FoxEdgeOperate(name = "读取温湿度", polling = true, type = FoxEdgeOperate.decoder, timeout = 2000)
    public static Map<String, Object> unpackReadInputRegister(String hexString, Map<String, Object> param) {
        return (Map<String, Object>) operateReadRegister(hexString, param);
    }

    /**
     * 编码/解码 读取保持寄存器
     *
     * @param hexString 解码报文：如果它为空，那么为编码操作，如果不为空，那么为编码操作
     * @param param     参数表
     * @return 编码为String，解码为Map<String, Object>
     */
    private static Object operateReadRegister(String hexString, Map<String, Object> param) {
        // 取出设备地址
        Integer devAddr = (Integer) param.get("devAddr");
        String modbusMode = (String) param.get("modbusMode");


        // 检查输入参数
        if (MethodUtils.hasEmpty(devAddr, modbusMode)) {
            throw new ProtocolException("输入参数不能为空:devAddr, modbusMode");
        }


        // 准备参数
        param.put(ModBusConstants.ADDR, devAddr);
        param.put(ModBusConstants.REG_ADDR, 0);
        param.put(ModBusConstants.REG_CNT, 6);
        param.put(ModBusConstants.MODE, modbusMode);

        // 选择对应的协议类型
        ModBusProtocol protocol = ModBusProtocolFactory.createProtocol(modbusMode);

        // 编码处理
        if (hexString.isEmpty()) {
            byte[] pack = protocol.packCmdReadRegisters4Map((byte) 0x04, param);
            return HexUtils.byteArrayToHexString(pack);
        }

        // 解码处理

        byte[] arrCmd = HexUtils.hexStringToByteArray(hexString);
        // 报文解析
        ModBusReadRegistersRespond respond = protocol.unPackCmdReadInputRegisters2Respond(arrCmd);
        if (respond == null) {
            throw new ProtocolException("报文格式不正确，解析失败！");
        }

        Map<String, Object> value = new HashMap<>();
        if (respond.getStatus().length < 2) {
            return value;
        }

        int status = respond.getStatus()[0];
        if (status < 10000) {
            value.put("温度", status * 0.1);
        } else {
            value.put("温度", (status - 10000) * -0.1);
        }

        status = respond.getStatus()[1];
        value.put("湿度", status * 0.1);

        return value;
    }
}

```

## 介绍
> 制造商：中盛科技

> 产品名称：温湿度采集模块

> 产品型号：ZS-SHT30-1T-1H-485

> 官网连接：http://www.zskjdg.com/Product/productChild?childId=2&ProductType=humiture

## 设备报文参数

### 设备通讯报文
#### 请求报文
```txt
01 04 00 00 00 06 70 08
```
#### 响应报文
```txt
01 04 0c 01 1f 01 c8 41 e5 b7 a4 42 36 c3 91 14 e6
```


### 设备操作请求
#### 请求参数
```json
  {
    "param": {
      "devAddr": 1,
      "modbusMode": "RTU",
      "operateName": "读取温湿度"
    },
    "timeout": 2000,
    "deviceName": "Win32 COM3",
    "deviceType": "温湿度采集模块(ZS-SHT30-1T-1H-485)",
    "operateMode": "exchange",
    "operateName": "读取温湿度"
  }
```

#### 响应参数
```json
  {
    "uuid": "afbfb1c6-ad59-451f-ae17-147b5390ba4b",
    "operateMode": "exchange",
    "deviceName": "Win32 COM3",
    "deviceType": "温湿度采集模块(ZS-SHT30-1T-1H-485)",
    "operateName": "读取温湿度",
    "param": {
      "devAddr": 1,
      "modbusMode": "RTU",
      "operateName": "读取温湿度"
    },
    "timeout": 2000,
    "record": true,
    "data": {
      "commStatus": {
        "commSuccessTime": 1689173276267,
        "commFailedCount": 0,
        "commFailedTime": 0
      },
      "value": {
        "status": {
          "温度": 28.6,
          "湿度": 45.300000000000004
        }
      }
    },
    "code": 200,
    "msg": ""
  }
```



