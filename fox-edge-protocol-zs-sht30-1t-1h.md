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

## 工具

``` 
推荐工具：
MODBUS调试助手
``` 

## 源码

- fox-edge-server-protocol-zs-sht30-1t-1h

[源码](https://gitee.com/fierce_wolf/fox-edge-server/tree/master/fox-edge-server-protocol/fox-edge-server-protocol-zs-sht30-1t-1h)


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

