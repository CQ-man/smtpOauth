# smtpOauth 
* 基于Python语言的smtp Oauth 连接China Office 365(或21V O365)的邮箱  
#Pyhton代码示例
```Python
import requests
import smtplib
import base64

# 定义发件人地址和密码以及收件人地址信息
username = 'S02@abc.com'
password = 'your username password'
recipient = 'recipient@abc.com'

# Get a token
url = 'https://login.partner.microsoftonline.cn/your tenant id/oauth2/v2.0/token'
data = {
    'grant_type': 'password',
    'client_id': 'your client id',
    'username': username,
    'password': password,
    'scope': 'https://partner.outlook.cn/.default',
    'client_secret': 'your client secret',
}
res = requests.post(url, data=data)
print("请求响应结果", res)
token = res.json().get('access_token')
print("访问令牌", token)

# 将username和token组合成SASL XOAUTH2 format
#对于Microsoft 365（或office 365）必须将 `^A`替换为``\x01`` ``\\x``转换为``%``
xoauth = "user=%s\x01auth=Bearer %s\x01\x01" % (username, token)
print("XOAUTH2格式", xoauth)

# base64编码
xoauth = xoauth.encode('ascii')
xoauth = base64.b64encode(xoauth)
print(xoauth)
xoauth = xoauth.decode('ascii')

#定义邮件主题内容等信息
msg = ("From: %s\r\nTo: %s\r\nSubject: %s\r\n\r\nSo happy to hear from you!"
       % (username, recipient, "Smtp Oauth With Python",))

#连接SMTP服务器并发送邮件
try:
    smtp_conn = smtplib.SMTP('smtp.partner.outlook.cn', 587)
    # smtp_conn.set_debuglevel(True)
    smtp_conn.set_debuglevel(2)
    smtp_conn.ehlo()
    smtp_conn.starttls()
    smtp_conn.ehlo()
    smtp_conn.docmd('AUTH', 'XOAUTH2 ' + xoauth)
    smtp_conn.sendmail(username, recipient, msg)
    smtp_conn.quit()
    print("邮件发送成功")
except smtplib.SMTPException as e:
    print("邮件发送失败", e)
```

