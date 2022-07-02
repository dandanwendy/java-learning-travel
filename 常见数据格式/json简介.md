JSON，全称avaScript Object Notation，即JavaScript 对象表示法，也是当下最流行的web前后端数据交互格式。作为一名后端开发人员，在设计restful接口，或使用postman测试接口的时候，经常和Json打交道，因此必须掌握json。

# json语法

### json语法

json语法中主要有以下几种元素

- key: value键值对是基本的数据单元

- 大括号 **{}** 保存对象
- 中括号 **[]** 保存数组，数组可以包含多个对象

对象的元素可以是数组，数组的元素也可以是对象。数据（键值对或数据元素）之间由逗号分隔，对象之间也由逗号分隔。

### json值

JSON 值可以是

- 数字（整数或浮点数）
- 字符串（在双引号中）
- 逻辑值（true 或 false）
- 数组（在中括号中）
- 对象（在大括号中）
- null

# 常见json数据

### 简单json对象

```json
// 字符串为值 
{
    "name": "菜鸟教程"
}

// 数字为值
{
    "age": 30
}

// bool作值
{
  "age": true
}

// null为值
{
    "age": null
}
```

### 多个键值对

对象可以包含多个名称/值对

```json
{
    "name": "runoob",
    "alexa": 10000,
    "site": null
}
```

### 嵌套json对象

JSON 对象中可以包含另外一个 JSON 对象

```json
{
    "name":"runoob",
    "alexa":10000,
    "sites": {
        "site1":"www.runoob.com",
        "site2":"m.runoob.com",
        "site3":"c.runoob.com"
    }
}
```

### 数组

```json
[ "Google", "Runoob", "Taobao" ]
```

### 数组元素为json

数组的元素也 可以是一个Json对象

```json
{
    "name": "网站",
    "num": 3,
    "sites": [
        {
            "name": "Google"
        },
        {
            "name": "Runoob"
        },
        {
            "name": "Taobao"
        }
    ]
}
```

### 数组作值的Json对象

对象属性的值可以是一个数组：

```json
{
    "name": "网站",
    "num": 3,
    "sites": [
        "Google",
        "Runoob",
        "Taobao"
    ]
}
```

### 嵌套数组

```json
{
    "name": "网站",
    "num": 3,
    "sites": [
        {
            "name": "Google",
            "info": [
                "Android",
                "Google 搜索",
                "Google 翻译"
            ]
        },
        {
            "name": "Runoob",
            "info": [
                "菜鸟教程",
                "菜鸟工具",
                "菜鸟微信"
            ]
        },
        {
            "name": "Taobao",
            "info": [
                "淘宝",
                "网购"
            ]
        }
    ]
}
```





