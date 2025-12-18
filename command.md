## 容器中没有curl命令，使用wget命令代替，例如
```
  wget -q -O - http://10.90.0.2
```

你想在没有curl的容器里，用wget命令来模拟curl的常见操作，这是容器环境中很常见的需求，因为很多轻量级镜像会默认只安装wget。

wget和curl的核心功能都是网络请求，但语法和参数有明显区别。下面我会按curl的常见使用场景，给出对应的wget实现方式，覆盖最常用的功能。

### 一、基础请求场景
#### 1. 简单GET请求（获取网页内容到控制台）
curl的写法：
```bash
curl https://www.example.com
```
wget的等价写法（输出内容到控制台，不保存文件）：
```bash
wget -qO- https://www.example.com
```
- `-q`：静默模式，不输出wget的日志信息
- `-O-`：将内容输出到标准输出（控制台），而不是保存为文件

#### 2. 保存请求结果到文件
curl的写法：
```bash
curl https://www.example.com > index.html
# 或
curl -o index.html https://www.example.com
```
wget的等价写法（默认就是保存文件，也可以指定文件名）：
```bash
# 默认保存为index.html（远程文件名）
wget https://www.example.com
# 自定义文件名
wget -O index.html https://www.example.com
```

### 二、带请求头/参数的场景
#### 1. 发送自定义请求头
curl的写法：
```bash
curl -H "Content-Type: application/json" -H "Authorization: Bearer token" https://www.example.com
```
wget的等价写法（用`--header`或`-H`参数，和curl一致）：
```bash
wget -qO- --header="Content-Type: application/json" --header="Authorization: Bearer token" https://www.example.com
# 简写-H也可以
wget -qO- -H "Content-Type: application/json" -H "Authorization: Bearer token" https://www.example.com
```

#### 2. 发送POST请求（带表单数据）
curl的写法：
```bash
curl -X POST -d "username=test&password=123" https://www.example.com/login
```
wget的等价写法：
```bash
wget -qO- --post-data="username=test&password=123" https://www.example.com/login
```

#### 3. 发送POST请求（带JSON数据）
curl的写法：
```bash
curl -X POST -H "Content-Type: application/json" -d '{"name":"test","age":20}' https://www.example.com/api
```
wget的等价写法：
```bash
wget -qO- --header="Content-Type: application/json" --post-data='{"name":"test","age":20}' https://www.example.com/api
```

### 三、高级场景
#### 1. 跟随重定向
curl默认跟随重定向，wget需要加参数：
```bash
# curl写法
curl https://www.example.com/redirect
# wget写法（-L 表示跟随重定向）
wget -qO- -L https://www.example.com/redirect
```

#### 2. 发送DELETE/PUT请求
curl用`-X`指定方法，wget用`--method`：
```bash
# curl DELETE请求
curl -X DELETE https://www.example.com/api/123
# wget DELETE请求
wget -qO- --method=DELETE https://www.example.com/api/123

# curl PUT请求（带数据）
curl -X PUT -H "Content-Type: application/json" -d '{"name":"new"}' https://www.example.com/api/123
# wget PUT请求（带数据）
wget -qO- --method=PUT --header="Content-Type: application/json" --body-data='{"name":"new"}' https://www.example.com/api/123
```
> 注意：wget的`--post-data`只能用于POST，PUT/DELETE需要用`--body-data`传递数据。

#### 3. 忽略SSL证书验证（测试环境常用）
```bash
# curl写法
curl -k https://self-signed.example.com
# wget写法
wget -qO- --no-check-certificate https://self-signed.example.com
```

### 总结
1. **基础GET请求**：wget用`-qO-`替代curl的默认输出，实现内容直出控制台。
2. **请求方法与数据**：POST用`--post-data`，PUT/DELETE用`--method`+`--body-data`，自定义头信息两者都可用`-H`。
3. **特殊场景**：跟随重定向加`-L`，忽略SSL证书加`--no-check-certificate`。

这些写法覆盖了curl在日常使用中的绝大多数场景，足以在只有wget的容器中完成网络请求操作。
