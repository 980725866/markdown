## 什么是.pem和.pk8文件
.pem
在android对apk签名的时候，.pem这种文件就是一个X.509的数字证书，里面有用户的公钥等信息，是用来解密的。文件格式里面不仅可以存储数字证书，还能存各种key。

.pk8
以.pk8为扩展名的文件，应该和PKCS #8是对应的，用来保存private key。

### 转换平台签名命令
```java
// https://github.com/getfatday/keytool-importkeypair
./keytool-importkeypair -k platform.jks -p platform -pk8 platform.pk8 -cert platform.x509.pem -alias platform
./keytool-importkeypair -k ./platform.keystore -p platform -pk8 platform.pk8 -cert platform.x509.pem -alias platform
```

### pem和pk8文件转jks
```java
// 1将pk8 转成pem 格式文件
openssl pkcs8 -in platform.pk8 -inform DER -outform PEM -out platform.priv.pem -nocrypt
// 2生成pk12 文件
openssl pkcs12 -export -in platform.x509.pem -inkey platform.priv.pem -out platform.pk12 -name android
// 3输出jks文件
keytool -importkeystore -destkeystore ${filename}.jks -srckeystore platform.pk12 -srcstoretype PKCS12 -srcstorepass ${password}
```


### jks转pem和pk8
```java
// 1
keytool -importkeystore -srckeystore ${filename}.jks -destkeystore ${xxx}.p12 -srcstoretype jks -deststoretype PKCS12
// 2
openssl pkcs12 -in ${xxx}.p12 -nodes -out ${xxx_all}.rsa.pem -password pass:password
// 3
openssl pkcs12 -in ${xxx}.p12 -nodes -nokeys -out ${xxx}.x509.pem -password pass:${password}
// 4
openssl pkcs12 -in ${xxx}.p12 -nodes -cacerts -out ${xxx}.rsa.pem -password pass:${password}
// 5
openssl pkcs8 -topk8 -outform DER -in ${xxx}.rsa.pem -inform PEM -out ${xxx}.pk8 -nocrypt

```
### 查看签名信息
```java
keytool -list -v -keystore ${filename}.jks
```
