```                                                                          
┌──(root㉿kali)-[~/桌面]
└─# openssl enc -base64 -d -in flag.enc -out flag1.enc                
                                                                             
┌──(root㉿kali)-[~/桌面]
└─# openssl enc -decrypt -in flag1.enc -inkey key.der -out fag.txt
enc: Use -help for summary.
                                                                             
┌──(root㉿kali)-[~/桌面]
└─# openssl pkeyutl -decrypt -in flag1.enc -inkey key.der -out flag.txt
                                                                             
┌──(root㉿kali)-[~/桌面]
└─# cat flag.txt
flag{D0nT_uS3_Th3_kN0w_n}                                                                             
┌──(root㉿kali)-[~/桌面]
└─# 
```
## 随机数

`openssl rand -out rand.dat -base64 32`

## 摘要

- 直接做摘要

`openssl dgst -sha1 -out dgst.dat plain.txt`

- 先做摘要，然后对摘要进行签名/验签
    
    - 签名摘要
        
        `openssl dgst -sha1 -sign priv.key -out sig.dat plain.txt`
        
    - 验签摘要
        
        `openssl dgst -sha1 -verify pub.key -signature sig.dat plain.txt`
        

## 对称加密

- 块加密
    
    - 加密
    
    `openssl enc -sm4 -K "0123456789abcdeffedcba9876543210" -iv "12345678123456781234567812345678" -e -in plain.txt -out encrypted.dat`
    
    - 解密
    
    `openssl enc -sm4 -K "0123456789abcdeffedcba9876543210" -iv "12345678123456781234567812345678" -d -in encrypted.dat -out decrypted.dat`
    
- 流加密
    
    - 加密
        
        `openssl enc -rc4 -a -K 0000000000000000 -in plain.dat -out encrypted.dat`
        
    - 解密
        
        `openssl enc -d -rc4 -a -K 0000000000000000 -in encrypted.dat -out decrypted.dat`
        

## 证书请求

- 生成证书请求
    
    - 交互式，需要手动输入一些证书项，如CN，email等
        
        `openssl req -key mysite.key -new -out mysite.req -sha1 -utf8`  
        `openssl req -key mysite.key -passin pass:123456 -new -out mysite.req -sha1 -utf8 (私钥带密码的情况)`
        
    - 非交互式，事先编辑好一个配置文件xxx.cnf
        
        - 不需要扩展项
            
            ```ini
            [ req ]distinguished_name        = req_distinguished_nameprompt                    = nostring_mask = utf8only#default, pkix, utf8only, nombstr [ req_distinguished_name ]C                         = CNST                        = SHL                         = ShanghaiO                         = AwesomeCompanyOU                        = SSL GroupCN                        = RSA_USERemailAddress              = rsa@user.com
            ```
            
        - 证书需要多个CN或OU的情况
            
            ```ini
            [ req ] distinguished_name     = req_distinguished_name prompt                 = no string_mask = utf8only#default, pkix, utf8only, nombstr  [ req_distinguished_name ] C                      = CN ST                     = SH L                      = Shanghai O                      = AwesomeCompany 1.OU                     = Dept.DEV 2.OU                     = SSL Group 1.CN                     = mytest 2.CN                     = mysite emailAddress            = ssl@test.com
            ```
            
        - 需要带扩展项
            
            ```ini
            [ req ]distinguished_name        = req_distinguished_nameprompt                    = noreq_extensions            = v3_reqstring_mask = utf8only#default, pkix, utf8only, nombstr [ req_distinguished_name ]C                         = CNST                        = SHL                         = ShanghaiO                         = AwesomeCompanyOU                        = SSL GroupCN                        = RSA_USERemailAddress              = rsa@user.com [ v3_req ]basicConstraints          = CA:FALSEkeyUsage                  = nonRepudiation, digitalSignature, keyEnciphermentsubjectAltName          = @alt_names [alt_names]email   = ssl@test.cnURI     = http://www.test.com/DNS.1   = *.test.cnDNS.2   = www.test.com.cnDNS.3   = www.test.com.cn:8443DNS.4   = 192.168.190.7:8443RID     = 1.2.4.6IP      = 192.168.1.8 dirName = mydir [ mydir ]CN    = aaaO     = bbb
            ```
            
        
        `openssl req -new -config test.cnf -key mysite.key -out mysite.req -sha1 -utf8`
        
        `openssl req -new -config test.cnf -key mysite.key -passin pass:123456 -out mysite.req -sha1 -utf8 (私钥带密码的情况)`
        
- 验证请求
    
    `openssl req -verify -in mysite.req -noout`
    
- 解析请求
    
    `openssl req -in mysite.req -text`
    

## 签发证书

- CA根证书
    
    `openssl x509 -req -days 3650 -in rootca.req -signkey rootca.key -out rootca.pem -passin pass:xxxx(私钥带密码的情况) -CAcreateserial`
    
- 用户证书
    
    `openssl x509 -req -in user.req -CA rootca.pem -CAkey rootca.key -out user.pem -passin pass:12345678 v3_req -CAcreateserial -days 3650`
    
- 生成v3证书，需要加配置文件指定扩展项
    
    `-extfile xxx.conf -extensions v3_req`
    

## 其他证书操作

- 证书解析
    
    `openssl x509 -in mysite.pem -noout -text`
    
- 证书转请求
    
    `-x509toreq`
    
- 克隆包含相同扩展项的证书
    
    `openssl x509 -certupdate -force_pubkey test.pub -in test.pem -CA DemoCA.pem -CAkey DemoCA.key -out test.pem.new -days 9999`
    
- 检查证书用途
    
    ```yaml
    $ openssl x509 -purpose -noout -in chen.pem	Certificate purposes:	SSL client : Yes	SSL client CA : No	SSL server : Yes	SSL server CA : No	Netscape SSL server : Yes	Netscape SSL server CA : No	S/MIME signing : Yes	S/MIME signing CA : No	S/MIME encryption : Yes	S/MIME encryption CA : No	CRL signing : Yes	CRL signing CA : No	Any Purpose : Yes	Any Purpose CA : Yes	OCSP helper : Yes	OCSP helper CA : No	Time Stamp signing : No	Time Stamp signing CA : No
    ```
    

## 验证证书链

`openssl verify -CAfile testca.pem mysite.pem`

完整返回OK，不完整返回类似`error 2 at 1 depth lookup:unable to get issuer certificate`

- 多级CA或多个证书链
    
    ```bash
    mkdir cacp ca-*.pem ca/   # 将所有CA证书复制到一个单独的目录c_rehash ca       # 创建所有CA证书的索引（基于DN项的lhash，此命令也是openssl自带的）openssl verify -CApath ca mysite.pem
    ```
    

## 非对称加密

- 签名
    
    `openssl pkeyutl -sign -inkey rsa.key -in rsa.dat -out sign.dat`
    
- 验签
    
    `openssl pkeyutl -verify -certin -inkey rsa.pem -in rsa.dat -sigfile sign.dat`
    
    `openssl pkeyutl -verify -inkey rsa.key -in rsa.dat -sigfile sign.dat`
    
- 加密
    
    `openssl pkeyutl -encrypt -inkey rsa.key -in rsa.dat -out enc.dat`
    
- 解密
    
    `openssl pkeyutl -decrypt -inkey rsa.key -in enc.dat -out source.dat`
    

## pkcs7

- 签名
    
    `openssl smime -sign -in short.dat -signer rsa.pem -inkey rsa.key -out rsa.sig -outform PEM -nodetach -binary -md sha256`
    
- 验签
    
    `openssl smime -verify -CAfile rsa-ca.pem -signer rsa.pem -in rsa.sig -inform PEM -noverify -content short.dat -binary`
    
- 加密
    
    `openssl smime -encrypt -sha1 -in long.dat -outform PEM -out rsa.env -binary rsa.pem`
    
- 解密
    
    `openssl smime -decrypt -in rsa.env -out rsa.plain -inkey rsa.key -inform PEM -binary`
    

## 密钥操作

- 生成密钥
    
    - genrsa
        
        `openssl genrsa -out rsa.key 2048(私钥不带密码)`
        
        `openssl genrsa -out rsa.key -aes256 -passout pass:123456 2048(私钥带密码)`
        
    - ecparam
        
        `openssl ecparam -name CN-GM-ECC -out sm2.param`
        
        `openssl ecparam -in sm2.param -out sm2.key -genkey -noout`
        
    - genpkey
        
        `openssl genpkey -algorithm RSA -out rsa.key -pkeyopt rsa_keygen_bits:2048`
        
        `openssl genpkey -parafile sm2.param -out sm2.key`
        
        ``
        
- 不带密码的私钥==>带密码的私钥
    
    `openssl rsa -in rsa.key -out xxx.key -aes256 -passout pass:123456`
    
    `openssl ec -in sm2.key -out xxx.key -sm4 -passout pass:123456`
    
- 带密码的私钥==>不带密码的私钥
    
    `openssl rsa -in xxx.key -passin pass:123456 -out yyy.key`
    
    `openssl ec -in xxx.key -passin pass:123456 -out yyy.key`
    
- pkey加解密私钥
    
    `openssl pkey -in rsa.key -out rsa_enc.key -des3 -passout pass:1234`
    
    `openssl pkey -in rsa_enc.key -out rsa.key -passin pass:1234`
    
- 从密钥对提取公钥
    
    `openssl rsa -in chen.key -pubout -out chen_pub.key`
    

## 格式转换

- PEM私钥转PKCS#8
    
    `openssl pkcs8 -topk8 -in mysite.key -out mysite.pk8 -outform PEM`
    
- PKCS#8转PEM
    
    `openssl rsa -in mysite.pk8 -out mysite.key`
    
- PEM转PKCS12
    
    `openssl pkcs12 -export -inkey mysite.key -in mysite.pem -nodes -out mysite.p12(输出不带口令的p12证书)`
    
    `openssl pkcs12 -export -inkey mysite.key -in mysite.pem -passout pass:123456 -out mysite.p12 (输出带口令的p12证书)`
    
- P12转证书
    
    `openssl pkcs12 -in mysite.p12 -nokeys -out mysite.pem`
    
    `openssl pkcs12 -in mysite.p12 -nokeys -passin pass:123456 -out mysite.pem (p12文件带口令的情况)`
    
- P12转私钥
    
    `openssl pkcs12 -in mysite.p12 -nocerts -nodes -out mysite.key (输出不加密的私钥)`
    
    `openssl pkcs12 -in mysite.p12 -nocerts -passout pass:123123 -out mysite.key (输出加密后的私钥)`
    
    `openssl pkcs12 -in mysite.p12 -nocerts -passin pass:123456 -passout pass:123123 -out mysite.key (p12文件带口令的情况)`
    

## 其他

- asn1解析
    
    `openssl asn1parse -dump -i -in xxx`
    
- 使用引擎
    
    `openssl xxx -engine myengine.so ...`