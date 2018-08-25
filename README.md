[中文](./GateSchema_zh.md)  

# GateSchema

**GateSchema** is a specification that **describes data structure and format**. It specifies a list of keywords and a JSON-based format to define constraints of data. 

With GateSchema, you may describe the Input/Output data format of the Interfaces, and use a validator to perform data validation.

A GateSchema instance consists of a list of **constraints**, which specify the existence, data types and other attributes of the Input.

Instead of writing raw JSON constraints, you may build a schema using an implementation, such as [gateschema-js](http://github.com/gateschema/gateschema-js), which provides simple syntax.

## Formation of Constraint

A constraint is a JSON object. It contains at least one **keyword** field. It may also contain an optional **msg** field to provide user-defined error message which will show up when the Input does not satisfy the schema. There could also be an optional **args** field if the keyword takes parameter(s). If a constraint has neither an args nor a msg, a keyword string can be used to describe the constraint directly.

Examples of constraint:

- Keyword String
```json
"required"
```

- Keyword only
```json 
{
    "keyword": "required"
}
```
- Keyword and msg
```json 
{
    "keyword": "required",
    "msg": "This field is required"
}
```


- Keyword, msg and arg
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

## GateSchema Example

This is an example of Gateschema which expects the Input that satisfy the following conditions:
- indispensable
- a map, that include
	- a required string 'name'
	- a optional number 'mobile'
	- a required string 'address' that should not be empty 
	
```json 
[
    "required",
    {
        "keyword": "map",
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

## Preset Keyword

A standard implementation in accordance with GateSchema specification should at lease support all the preset keywords.

### `required`
 
The Input must be present and not null.


```json 
"required"
```

### `optional`

The Input is optional.

```json 
"optional"
```

### `boolean`

The Input is boolean.

```json 
"boolean"
```

### `binary`  

The Input is a binary.

```json 
"binary"
```

### `number`   

The Input is a number.

```json 
"number"
```

### `string`  

The Input is a string.

```json 
"string"
```

### `any`

The Input accepts any data type.

```json 
"any"
```

### `enum(definition: {[label: string]: number})`

The Input value should be one of the predefined values.

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

The Input is a list of values, each of which satisfies the given schema.

```json 
{
    "keyword": "list",
    "args": [
        "number"
    ]
}
```

### `map(definition: {[key:string]: GateSchema})`

The Input is supposed to be a map with keys and values that satisfy the definition.

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

The Input value should satisfy one of the given schemas.


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

The Input value should equal to the specified value.

```json 
{
    "keyword": "value",
    "args": [
        5
    ]
}
```

### `switch(path: string, cases: {case?: GateSchema, schema: GateSchema}[])`

Expect the Input value to satisfy a schema which depends on the value at the given path. Simply put, if the value at the given path matches a case(e.g. case A), the Input should satisfy the schema defined in case A.
If there is no matching case, it will pass the check.
You may use the keyword 'any' to specify a default case.

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

Expect the Input value to be equal to the value at the given path.


```json 
{
    "keyword": "equal",
    "args": [
        "/password"
    ]
}
```

### `format(type: "date" | "date-time" | "hostname" | "uri" | "url" | "email" | "ipv4" | "ipv6")` 

Expect the Input value to be in the specified format (including "date", "date-time", "hostname", "uri", "url", "email", "ipv4", "ipv6").

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


The length of the Input should accord with the specified `range`. 
`range` can be a fixed `number` or an array `[min?: number, max?: number]` which specifies the range of the length.
If the Input value is a `string`, the `range` refers to the number of characters. 
If the Input value is a `list`, the `range` refers to the number of elements. 
If the Input value is a `binary`, the `range` refers to the number of bytes. 

```json 
{
    "keyword": "length",
    "args": [
        [6, 16]
    ]
}
```

### `not(schema: GateSchema)`  

This keyword is used to exclude the specific schema which the Input is not supposed to satisfy.

```json 
{
    "keyword": "not",
    "args": [
        "string"
    ]
}
```

### `notEmpty`
The Input should not be empty. It could neither be 0, '', `list` without elements nor `map` without any key.

```json 
{
    "keyword": "notEmpty"
}
```

### `pattern(regex: string | RegExp, flags?: string)`

The Input should satisfy a regular expression.

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

Expect each element in the `list` is unique.

```json  
"unique"
```

### `other(...args: any[])`

This keyword is used to store additional information for other purposes, such as generating data and UI rendering.

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
