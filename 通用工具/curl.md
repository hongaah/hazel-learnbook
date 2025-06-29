# curl

Client Url

CURL 是一个利用 URL 语法在命令行下工作的文件传输工具

[curl 的用法指南](https://www.ruanyifeng.com/blog/2019/09/curl-reference.html)

## 应用

### 将浏览器的请求转换为 CURL 命令

浏览器端，找一个请求，右击复制格式，会有一个以 curl 格式复制，然后可以直接粘贴到浏览器控制台或终端发送请求，什么请求都可以发，不需要权限，因为复制的时候已经把关键信息带上来

有时候浏览器的请求你觉得不对了，可以直接把这个curl 发给后端

### 使用命令行往数据库读取数据

```bash
# 不带任何参数就是一个 get 请求 
$ curl www.baidu.com

# -d:post方式
$ curl -d 'name=hong&age=22' http://localhost:3000/users/addPerson
```

### 实现某些功能

yarn 版本升级：
$ curl -o- -L https://yarnpkg.com/install.sh | bash
