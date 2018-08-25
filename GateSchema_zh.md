[English](./README.md)  

# GateSchema   

**GateSchema** 是一个**描述数据的结构和格式**的规范。它制定 了一系列的关键字，并规定了一种基于 JSON 的格式，用来描述数据的约束。

一个 GateSchema 实例包含一个 **约束(constraint)** 列表，用来表达输入的数据是否必须存在，应有的数据类型等。

我们建议使用具体实现去构建 schema，例如 [gateschema-js](http://github.com/gateschema/gateschema-js)，它提供了简便的语法。

GateSchema 可以用于描述接口的输入或输出，然后使用验证器验证输入或输出是否满足要求。


## 约束的构成

一个约束是一个 JSON 对象，它至少包含一个 **关键字(keyword)** 字段。它还可以包含一个可选的 **消息(msg)** 字段，用来表示输入不满足该约束时的**自定义错误消息**。当关键字带有参数时，还需要包含 **参数(args)** 字段。如果一个约束不包含 args ，也没有 msg 的时候，可以直接使用 keyword 字符串表示。


下面是一些约束的示例：

- Keyword 字符串

```json
"required"
```

- 只包含 keyword 关键字
```json 
{
    "keyword": "required"
}
```
- 包含 keyword 和 msg
```json 
{
    "keyword": "required",
    "msg": "This field is required"
}
```
- 包含 keyword、msg 和 args
```json 
{
    "keyword": "list",
    "msg": "This field should be a list of number",
    "args": [
        [
            {
                "keyword": "number"
            }
        ]
    ]
}
```


## GateSchema 示例

下面是一个 GateSchema 的例子，它期望的输入必须满足以下条件：
- 必须存在
- 数据类型为 map，并且包含
    * 一个必须存在的、字符串类型的 `name` 字段
    * 一个可选的、数字类型的`mobile`字段
    * 一个必须存在的、不可为空的、字符串类型的`address`字段
```json 
[
    "required",
    {
        "keyword": "map"
        "args": [
            {
                "name": ["required", "string"],
                "mobile": ["optional", "number"],
                "address": ["required", "string", "notEmpty"]
            }
        ]
    }
]

```  

## 内置 keyword    

一个符合此规范的标准实现至少应该支持所有的内置 keyword。

### `required`

表示 input 是一个必须存在的值, 且不能为 null。

```json 
"required"
```

### `optional`

表示 input 是一个可选的值。

```json 
"optional"
```

### `boolean`
表示 input 是布尔类型。

```json 
"boolean"
```

### `binary`  
表示 input 是一个二进制数。

```json 
"binary"
```

### `number`   

表示 input 是数字类型。  

```json 
"number"
```

### `string`  

表示 input 是字符串类型。

```json 
"string"
```

### `any`

表示 input 可以是任意类型的值。

```json 
"any"
```

### `enum(definition: {[label: string]: number})`

表示 input 是一个枚举类型。

```json 
{
    "keyword": "enum",
    "args": [
        {
            "MALE": 0,
            "FEMALE": 1
        }
    ]
}
```

### `list(schema: GateSchema)`

表示 input 是一个数组类型，其每一个元素都必须满足给定的 schema。 

```json 
{
    "keyword": "list",
    "args": [
        "number"
    ]
}
```

### `map(definition: {[key:string]: GateSchema})`

表示 input 是 map 类型。

```json 
{
    "keyword": "map",
    "args": [
        {
            "name": ["required", "string"],
            "phone": ["required", "number"]
        }
    ]
}
```


### `oneOf(schemas: GateSchema[])`

表示 input 满足给定的若干 schema 中的一种。

```json 
{
    "keyword": "oneOf",
    "args": [
        ["number"],
        ["string"]
    ]
}
```

### `value(value: any)`

表示 input 是一个具体值。

```json 
{
    "keyword": "value",
    "args": [
        5
    ]
}
```

### `switch(path: string, cases: {case?: GateSchema, schema: GateSchema}[])`

表示 input 需要满足的约束依赖于给定的 path 里的值。也就是说，如果给定的 path 的值符合某个 case，则 input 需要满足该 case 里的 schema 的约束。
如果没有匹配的 case，则无需满足任何约束。
此外，可以使用 any 关键字来表示默认 case。

```json 
{
    "keyword": "switch",
    "args": [
        "/phone"
        [
            {
                "case": ["required"],
                "schema": ["required"]
            }
        ],
        [
            {
                "case": ["any"],
                "schema": ["optional"]
            }
        ]
    ]
}
```

### `equal(path: string)`

表示 input 值与给定路径里的值相等。

```json 
{
    "keyword": "equal",
    "args": [
        "/password"
    ]
}
```

### `format(type: "date" | "date-time" | "hostname" | "uri" | "url" | "email" | "ipv4" | "ipv6")` 

表示 input 值必须符合给定的格式（包括 "date"，"date-time"，"hostname"，"uri"，"url"，"email"，"ipv4"，"ipv6"）

- 举例如下：

* date: see full-date in https://tools.ietf.org/html/rfc3339#section-5.6, examples:  
  * 1990-12-31  
  * 2018-07-28   
* date-time: see date-time in https://tools.ietf.org/html/rfc3339#section-5.6, examples: 
  * 1990-12-31T15:59:59+02:00
  * 1990-12-31T15:59:59-08:00
  * 2017-07-21T17:32:28Z
* hostname: examples:  
  * github.com
  * a-n-y.sub.123.domain
  * localhost
  * 8.8.8.8
* uri: expamles:  
  * https://github.com
  * https://github.com/gateschema
  * https://github.com:443/gateschema?query=any
  * schema://path:port?query'
* url: examples:  
  * ftp://github.com
  * http://github.com/gateschema
  * https://github.com:443/gateschema?query=any
* email: see http://www.w3.org/TR/html5/forms.html#valid-e-mail-address, examples:  
  * test@github.com
  * test@localhost
  * 123@github.com
  * t12@github.com
* ipv4: examples:  
  * 8.8.8.8
  * 192.168.1.1
* ipv6: examples: 
  * 2001:0db8:0000:0000:0000:ff00:0042:1234
  * 2001:db8:0:0:0:ff00:42:5678
  * 2001:db8::ff00:42:8765

```json 
{
    "keyword": "format",
    "args": [
        "email"
    ]
}
```

### `length(range: number | [number] | [undefined, number] | [number, undefined] | [number, number])`

表示 input 的长度信息，它带有一个 range 参数。range 既可以是数字，用来表示一个固定的长度，也可以是数组，用来表示允许的长度范围。

```json 
{
    "keyword": "length",
    "args": [
        [6, 16]
    ]
}
```

### `not(schema: GateSchema)`  

表示 input 不可以是给定的 schema 的样式。

```json 
{
    "keyword": "not",
    "args": [
        "string"
    ]
}
```

### `notEmpty`

表示 input 不可以是 0，空的 string，空的 list，或者是没有任何键值的 map。

```json 
{
    "keyword": "notEmpty"
}
```

### `pattern(regex: string | RegExp, flags?: string)`

表示 input 必须满足一个正则表达式。

```json 
{
    "keyword": "pattern",
    "args": [
        "ab+c",
        "i"
    ]
}
```

### `unique`

表示 list 中的每个元素都必须是独一无二的，不可以有重复的项。

```json  
"unique"
```

### `other(...args: any[])`

通过这个 other 关键字，用户可以储存一些额外的信息，如生成数据、生成表单等。  

```json  
{
    "keyword": "other",
    "args": [
        "form",
        {
            "component": "InputNumber"
        }
    ]
}
```

 
 
