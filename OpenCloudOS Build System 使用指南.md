# <center> OpenCloudOS Build System 使用指南
## 账号获取
- 开发者账号获取
	- 联系@maxonxie @haifulian
- Build系统账号&权限
	- 联系 @victorzhong
- 认证配置
```
dnf install -y krb5-libs 

vim /etc/krb5.conf

#File modified by ipa-client-install
  
includedir /etc/krb5.conf.d/

[libdefaults]
  default_realm = OPENCLOUDOS.TECH
  dns_lookup_realm = false
  dns_lookup_kdc = false
  rdns = false
  dns_canonicalize_hostname = false
  ticket_lifetime = 24h
  forwardable = true
  udp_preference_limit = 0


[realms]
  OPENCLOUDOS.TECH = {
    kdc = freeipa.opencloudos.tech:88
    master_kdc = freeipa.opencloudos.tech:88
    admin_server = freeipa.opencloudos.tech:749
    kpasswd_server = freeipa.opencloudos.tech:464
    default_domain = opencloudos.tech
    pkinit_anchors = FILE:/var/lib/ipa-client/pki/kdc-ca-bundle.pem
    pkinit_pool = FILE:/var/lib/ipa-client/pki/ca-bundle.pem

  }


[domain_realm]
  .opencloudos.tech = OPENCLOUDOS.TECH
  opencloudos.tech = OPENCLOUDOS.TECH
  build.opencloudos.tech = OPENCLOUDOS.TECH
```

#Build 系统使用
## 1. 安装build cli 工具
```
 sudo dnf install koji
```
## 2. 配置buildserver 地址
```
# cat /etc/koji.conf
[koji]
;configuration for koji cli tool
;server = http://hub.example.com/kojihub
server = https://build.opencloudos.tech/kojihub
weburl = https://build.opencloudos.tech/koji
topurl = https://build.opencloudos.tech/kojifiles/

;the principal to auth as for automated clients
principal = $username@OPENCLOUDOS.TECH  ## 这里需要开发者情况填写
;the keytab to auth as for automated clients
;keytab = /root/kojiadmin.keytab

; fedora uses kerberos auth
authtype = kerberos
use_fast_upload = yes
```
## 3. 获取证书session（有效期24小时）
  ```
 kinit username
  ```
## 4. 测试cli和build系统的链接状态
```
koji hello
```
## 5. 发起一次build
 - **发起一次scratch build**  （测试提交，just build）， 强烈建议new packager 同学执行这个步骤。
  ```
  koji build --nowait dist-oc8  --scratch 'git+https://gitee.com/src-opencloudos-rpms/qt5-qtbase?#790d0333bf477a03004c24c33f4326caef8b107e'
	or(建议上面的方式)
  koji build --nowait dist-oc8  --scratch 'git+https://gitee.com/src-opencloudos-rpms/qt5-qtbase?#origin/oc8'
  ```
  `--nowait` 非交互模式
  `dist-oc8` 构建目标，其他目标 https://build.opencloudos.tech/koji/buildtargets
  `--scratch` 发起的build 是一次测试提交，build系统仅发起build流程，build结果不会收录到对应的仓库和tag。
  `790d0333bf477a03004c24c33f4326caef8b107e`  gitee 仓库commitId，总是指向最新的commitId。
  `'git+https://gitee.com/src-opencloudos-rpms/'` 项目仓库地址，格式如此。
  `qt5-qtbase` 软件包仓库名称，和gitee 仓库一致。
  
 - **发起一次正式 Build**（If success, Build系统会将这个包收录等待发布到测试Yum仓库）
  ```
  koji build --nowait dist-oc8  'git+https://gitee.com/src-opencloudos-rpms/qt5-qtbase?#790d0333bf477a03004c24c33f4326caef8b107e'
  ```
