# 登录密码及授权密钥简配，限制与封禁IP，并锁Root

## SSH密钥

### 客户端

本机会将这一登录信息保存在`~/.ssh/known_hosts`文件当中，再次登录到远程服务器不用输入密码。

```
# 参数说明：-t 指定要创建的类型；-b 密钥长度；-f 指定文件名；id_rsa-remotessh 名字随意
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa-remotessh
```

"-i"是指定公钥文件上传到服务器。

```
ssh-copy-id -i .ssh/id_rsa.pub user@server
```

输入密码，至此客户端完成操作，下面是服务器端，即远程主机的配置。

### 服务端

编辑 /etc/ssh/sshd_config 文件，添加如下设置：

```
# 是否允许Public Key 
PubkeyAuthentication yes
# 允许Root登录
PermitRootLogin yes
# 设置是否使用口令验证。
PasswordAuthentication no # no 代表任何人远程访问都只能通过密钥，除非去机房或VNC屏幕远程
```

重启SSH服务，`systemctl restart sshd.service`。

#### 设置ssh路径下的权限

```
chmod 700 /home/xxx && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
```

## 密码简化


配置密码策略，修改密码得像4位数的验证码一样简单。vi /etc/pam.d/system-auth

```
# 密码验证三次，什么大小写特殊字符统统给我去掉
password requisite pam_pwquality.so try_first_pass local_users_only retry=3
password requisite pam_pwquality.so authtok_type= lcredit=0 ucredit=0 dcredit=0 ocredit=0  minlen=4
```

## 限定公司与家的IP

## fail2ban

下载安装设置自启与启动fail2ban。

```
yum install -y fail2ban && systemctl enable fail2ban.service
```

配置 `vi /etc/fail2ban/jail.conf`

```
# 注意时区问题：systemctl restart rsyslog
# 注意端口号：我们修改ssh端口后，fail2ban也需要修改端口号
action = iptables[name=SSH,port=ssh,protocol=tcp] 
enabled = true
filter = sshd
logpath = /var/log/secure   #日志位置
bantime =  800              #封锁时间长达一月以上（24*30）
maxretry = 2                #失败2次即封禁
findtime = 43200            #12小时之内(60*60*12)
# 可以定制化发送邮件
sendmail-whois[name=SSH, dest=your@email.com, sender=fail2ban@example.com,sendername="Fail2Ban"]    
```

启动服务 `systemctl start fail2ban.service`，fail2ban开始生效。

```
systemctl restart fail2ban
```

日志查看 `cat /var/log/fail2ban.log`


***power by 老子他妈的就爱简单密码。***