# 域名证书申请和更换

一级域名com,edu,cn,net,org

二级域名lwork.com   lwork.cn

\#\#\# 泛域名证书的使用,\*\*只支持2级域名\*\*

\*\*\*.lwork.com只支持bac.lwork.com,abc.lwork.com不支持abc.bac.lwork.com\*\*

\*\*1.更换证书\*\*

域名证书到期，到godaddy对应域名下载新证书，因为使用的是nginx

所以下载类别选其他



证书文件

dfe616168a014563.crt

dfe616168a014563.pem

gd\_bundle-g2-g1.crt

生成新的crt

\`cat dfe616168a014563.crt gd\_bundle-g2-g1.crt &gt; lwork.crt\`

使用该crt替换老的crt

----------

\*\*2.更换证书\*\*

起因是最近需要更换一台服务器上的SSL证书，客服是直接把crt包（含pem和2个crt，\*\*godaddy下载的apache版本\*\*包）、一个key.txt和一个csr.txt发给我，这次不大一样，但是文件应该没问题，我nginx的配置使用的是key+crt形式

正常合并crt

\`cat 4babe335689330b5.crt gd\_bundle-g2-g1.crt &gt; private\_cloud2.crt\`

key使用客服发过来的key.txt

nginx -t 报错 \(SSL: error:0906D06C:PEM routines:PEM\_read\_bio:no start line:Expecting: ANY PRIVATE KEY\)

网上找到一种思路

\`\`\`

file xxx.key \#查看编码格式  

iconv -l \#查看系统支持的格式  

vi xxx.key 

:set fileencoding=ASCII \#保存退出，确认一遍格式

\`\`\`

nginx -t成功

\[https://stackoverflow.com/questions/43729770/nginx-godaddy-ssl/43730023\]\(https://stackoverflow.com/questions/43729770/nginx-godaddy-ssl/43730023\)

\*\*3. 新域名申请证书\*\*

\[https://sg.godaddy.com/zh/help/nginx-csr-3601\]\(https://sg.godaddy.com/zh/help/nginx-csr-3601\)

1.  通过 SSH 连接至您的服务器（\[了解详细信息\]\(https://sg.godaddy.com/zh/help/ssh-4943\)）。

2.  运行以下命令：

    

    openssl req -new -newkey rsa:2048 -nodes -keyout\*您的域名\*.key -out\*您的域名\*.csr

    

    用您保护的域名替换\*\`您的域名\`\*。 例如，如果您的域名是 coolexample.com，您可以键入\`coolexample.key\`和\`coolexample.csr\`。

    

3.  输入申请的信息：

    

    \| 字段 \| 输入的内容 \|

    \| --- \| --- \|

    \| \*\*通用名称\*\* \| 您想要保护的完全合格的域名，或 URL。  

    如果您申请通配证书，在您想要通配的通用名称左侧添加一个星号 \(\\*\)，例如\*\\*.coolexample.com\*。 \|

    \| \*\*组织\*\* \| 您企业的法定注册名称。 如果以个人身份注册，则输入证书申请者的姓名。 \|

    \| \*\*组织单位\*\* \| 如适用，请输入营业名称 \(Doing Business As, DBA\)。 \|

    \| \*\*城市或地区\*\* \| 您的组织注册/所在的城市名称。 不要缩写。 \|

    \| \*\*州或省\*\* \| 您的组织所在的州或省的名称。 不要缩写。 \|

    \| \*\*国家\*\* \| 您的组织合法注册所在国家的\[国家代码\]\(http://www.iso.org/iso/home/standards/country\_codes/iso-3166-1\_decoding\_table.htm\)（按照国际标准化组织 \(ISO\) 的双字母格式）。 \|

    \| \*\*密码\*\* \| （\*可选\*）： SSL 的密码。 如果您将此字段留空，则 SSL 无密码，会让您暴露于其他风险中。 \|

    

4.  在文本编辑器中打开 CSR，然后复制所有文本。

5.  将完整的 CSR 粘贴到您账户的 SSL 申请区域。



