# TypeApi

一种以类型为核心的基于 JSON 的类 REST API 描述文档

并不能描述所有 REST API 情况也不打算那么做

## 文件根

```ts
{
    meta?: 元信息描述对象,
    root: 命名空间根描述对象,
    type: 类型定义对象[],
}
```

## 元信息描述

```ts
{
    name?: string,
    version?: string,
    doc?: string,
}
```

## 命名空间描述

- 命名空间根

  ```ts
  {
      api: 接口描述对象,
      items: 命名空间描述对象[],
  }
  ```

- 命名空间

  ```ts
  {
      name: string | 名称分部描述对象[],
      doc?: string,
      api: 接口描述对象,
      items: 命名空间描述对象[],
  }
  ```

  - 名称分部

    ```ts
    | string
    | {
          name: string,
          type?: 类型描述对象,
      }
    ```

- 接口

  ```ts
  {
      // Http 方法
      [method: string]: 接口详情,
  }
  ```

- 接口详情

  ```ts
  {
      doc?: string,
      headers?: HTTP 头对象,
      query?: URL 查询对象,
      body?: 请求体对象,
      res: 响应体对象,
      errs?: 错误响应对象[],
  }
  ```

  - HTTP 头对象

    ```ts
    {
        [成员名: string]: object 成员描述对象,
    }
    ```

  - URL 查询对象

    ```ts
    {
        // 指示遇到数组时如何处理，默认使用 json
        // `repeat` | `[][]` 仅限第一层数组，第二层永远是 json
        // `T,T` | `,T,T,` 仅限基础类型
        array?: 'json' | 'repeat' | 'repeat[]' | '[n]' | `[][]` | 'T,T' | ',T,T,' ,
        type: {
            [成员名: string]: object 成员描述对象,
        },
    }
    ```

    <details>
    <summary>array 格式示例</summary>

    - `json`
      ```
      ?foo=[1,2,3]
      ```
    - `repeat`
      ```
      ?foo=1&foo=2&foo=3
      ```
    - `repeat[]`
      ```
      ?foo[]=1&foo[]=2&foo[]=3
      ```
    - `[n]`
      ```
      ?foo[0]=1&foo[1]=2&foo[2]=3
      ```
    - `[][]`
      ```
      ?foo[]=1&foo[][]=2&foo[][][]=3
      ```
    - `T,T`
      ```
      ?foo=1,2,3
      ```
    - `,T,T,`
      ```
      ?foo=,1,2,3,
      ```

    </details>

  - 请求体对象

    不支持单接口多种格式，不考虑支持过时的 XML

    ```ts
    & {
          mime?: string | string[],  // 要求的 MIME 类型，将包含在请求头中，数组将包含任意一个
      }
    & (
      | {
            kind: 'json',
            type: 类型描述对象,
        }
      | {
            kind: 'query',
            query: URL 查询对象,
        }
      | {
            kind: 'text' | 'buffer' | 'blob',
        }
      | {
            kind: 'form',  // 将发送一个 FormData
            type: {
                [成员名: string]: object 成员描述对象,
            },
        }
    )
    ```

  - 响应体对象

    ```ts
    & {
          doc?: string
          mime?: string | string[],
      }
    & (
      | {
            kind: 'json',
            type: 类型描述对象,
        }
      | {
            kind: 'text' | 'buffer' | 'blob',
        }
    )
    ```
  - 错误响应对象

    ```ts
    & {
          code: number,
          doc?: string
          mime?: string | string[],
      }
    & (
      | { },
      | {
            kind: 'json',
            type: 类型描述对象,
        }
      | {
            kind: 'text' | 'buffer' | 'blob',
        }
    )
    ```

## 数据类型

所有类型描述对象都有 `id` 字段

### 基础类型

```ts
{
    kind: 'basic',
    ... 其他字段
}
```

- `kind`  
  是哪种类型，区分类型描述格式
- `type`  
  对应的 JS/TS 类型
- `id`  
  基础类型的唯一标识
- `json`  
  对应的 JSON 类型
- `signed`  
  是否有符号
- `size`  
  类型的大小

表格第一列是名称，后面是类型描述对象的成员值

| 名称             | `type`        | `json`    | `id`       | `signed` | `size` |
| ---------------- | ------------- | --------- | ---------- | -------- | ------ |
| 8 位整数         | `number`      | `number`  | `i8`       | `true`   | `8`    |
| 16 位整数        | `number`      | `number`  | `i16`      | `true`   | `16`   |
| 32 位整数        | `number`      | `number`  | `i32`      | `true`   | `32`   |
| 64 位整数        | `string`      | `string`  | `i64`      | `true`   | `64`   |
| 128 位整数       | `string`      | `string`  | `i64`      | `true`   | `128`  |
| 8 位无符号整数   | `number`      | `number`  | `u8`       | `false`  | `8`    |
| 16 位无符号整数  | `number`      | `number`  | `u16`      | `false`  | `16`   |
| 32 位无符号整数  | `number`      | `number`  | `u32`      | `false`  | `32`   |
| 32 位无符号整数  | `string`      | `string`  | `u64`      | `false`  | `64`   |
| 128 位无符号整数 | `string`      | `string`  | `u128`     | `false`  | `128`  |
| 大整数           | `string`      | `string`  | `bigint`   | `true`   |        |
| 16 位浮点数      | `number`      | `number`  | `f16`      | `true`   | `116`  |
| 32 位浮点数      | `number`      | `number`  | `f32`      | `true`   | `32`   |
| 64 位浮点数      | `number`      | `number`  | `f64`      | `true`   | `64`   |
| 十进制数         | `number`      | `number`  | `decimal`  | `true`   |        |
| 大数字           | `string`      | `string`  | `bignum`   | `true`   |        |
| 字符串           | `string`      | `string`  | `str`      |          |        |
| 字符             | `string`      | `string`  | `char`     |          |        |
| 布尔             | `boolean`     | `boolean` | `bool`     |          |        |
| 日期             | `string`      | `string`  | `date`     |          |        |
| 时间             | `string`      | `string`  | `time`     |          |        |
| 日期时间         | `Date`        | `string`  | `datetime` |          |        |
| UUID             | `string`      | `string`  | `uuid`     |          |        |
| 二进制数据       | `Blob`        |           | `blob`     |          |        |
| 文件             | `Blob`        |           | `file`     |          |        |
| 二进制数组       | `ArrayBuffer` |           | `buffer`   |          |        |

### 复合类型

- 引用

  ```ts
  {
      kind: 'ref',
      ref: '引用名',
      params?: 类型描述对象[],       // 可选字段，描述泛型参数
  }
  ```

- 数组

  数组类型的字段如下

  ```ts
  {
      kind: 'array',
      item: 类型描述对象,
  }
  ```

- 对象

  ```ts
  {
      kind: 'object',
      items: {,
          [成员名: string]: 成员描述对象,
      }
  }
  ```

  - 成员描述对象

    ```ts
    {
        type: 类型描述对象,
        optional?: boolean,          // 是否是可选的，就是 ts 的 ?:
        doc?: string,
    }
    ```

- 联合

  ```ts
  {
      kind: 'union',
      items: 类型描述对象[],
  }
  ```

- 合并

  ```ts
  {
      kind: 'merge',
      items: 类型描述对象[],
  }
  ```

- 映射

  对应 JS 的 `Map<A, B>`

  ```ts
  {
      kind: 'map',
      from: 类型描述对象,
      to: 类型描述对象,
  }
  ```

- 集合

  对应 JS 的 `Set<T>`

  ```ts
  {
      kind: 'set',
      type: 类型描述对象,
  }
  ```

## 类型定义

类型定义不属于类型描述对象

- 别名

  ```ts
  {
      kind: 'type',
      name: string,
      doc?: string,
      params?: string[],        // 泛型参数定义
      type: 类型描述对象,
  }
  ```

- 枚举

  ```ts
  {
      kind: 'enum',
      name: string,
      doc?: string,
      flags?: boolean,          // 标注是否是标志位枚举
      items: 枚举成员描述对象[],
  }
  ```

  - 枚举成员

    ```ts
    {
        name: string,
        value: number | string,
    }
    ```
