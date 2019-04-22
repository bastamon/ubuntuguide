# 第3章 打造命令工具
### 3.1.1 使用sys.argv获取命令行参数
```
from __future__ import print_function
import os
import sys

def main():
    sys.argv.append("")
   print(sys.argv[0])
    filename = sys.argv[1]
    if not os.path.isfile(filename):
        raise SystemExit(filename + 'dose not exists')
    elif not os.access(filename, os.R_OK):
        raise SystemExit(filename + 'is not accessible')
    else:
        print(filename + ' is accessible')        
      
        
if __name__ == '__main__':
    main()
```


可以得出语法:
```python 参数0(sys.argv[0]) 参数1(sys.argv[1]) …… 参数n(sys.argv[n]) ```  

---------------
补充：

在python2.x中 ，异常是这样的处理的，异常基类后面加一个逗号“ ，”  然后跟着异常类型:
```
import traceback
try:
  1/0
except Exception , err:
  print err
```

而在python3.x中，异常是这样处理的，基类通过关键 词"as" 连接异常类型:
```
import traceback
try:
  1/0
except Exception as err:
  print(err)
```
    
### 3.1.2使用sys.stdin和fileinput读取标准输入
>文件“read_stdin.py”：
```
from __future__ import print_function
import sys

for line in sys.stdin:
    print (line,end="")
```

就可以像shell一样，在终端输入，注意第二行的实现方式：
```
$ cat /etc/passwd | python read_stdin.py    //  cat读管道方式交给read_stdin.py
$ python read_stdin.py < /etc/passwd    //read_stdin.py实现读地址
$ python read_stdin.py -    //read_stdin.py实现读标准键盘输入
```

------------------------------------
调用read函数读取标准输入中的所有内容调用readlines函数将标准输入的内容读取到一个列表中（不太懂）：
```
from __future__ import print_function
import sys

def get_content():
    return sys.stdin.readlines()

print(get_content())
```

-------------------------------------


fileinput可以依次读取命令行参数中给出的多个文件，也就是说fileinput会遍历sys.argv[1:]列表，fileinput遍历文件内容:
```
#!/usr/bin/python
from __future__ import print_function
import fileinput
for line in fileinput.input():
    meta= [fileinput.filename(),fileinput.fileno(),fileinput.filelineno(),fileinput.isfirstline(),fileinput.isstdin()]
    print(*meta,end="")
    print(line,end="")
```

使用shell运行：
```
$ cat /etc/passwd | python read_from_fileinput.py
$ python read_from_fileinput.py < /etc/passwd
$ python read_from_fileinput.py /etc/passwd
$ python read_from_fileinput.py /etc/passwd /etc/hosts
```
### 3.1.3 使用SystemExit异常打印错误信息
>文件“tes_stdout_stderr.py”
```
import sys

sys.stdout.write('hello') 
sys.stderr.write('world') 
```

调用：
```
$ python tes_stdout_stderr.py >/dev/null
$ python tes_stdout_stderr.py 2>/dev/null
```
>文件：test_system_exit.py
```
import sys

sys.stderr.write('error message')
sys.exit(1)
```

shell输入：
```
python test_system_exit.py
```

### 3.1.4使用getpass获取密码

实现隐藏标准输入
```
from __future__ import print_function
import getpass

user=getpass.getuser()
passwd=getpass.getpass('your passwd:')
print(user,passwd)
```

## 3.2 使用ConfigParse解析配置文件☆☆☆(dependancy python 2.7~2.8)

假设已安装MySql，那就有/etc/mysql/my.cnf,pip的配置文件位于~/.pip/pip.conf中。还有*.ini等<font color=#FF1493>配置文件</font>

ConfigParse中判断配置项相关的方法有：
* sections:返回一个包含所有章节的列表
* has_section:判断章节是否存在
* items:以元组的形式返回所有选项
* options:返回一个包含章节下所有选项的列表
* has_option:判断某个选项是否存在
* get、getboolean、getinit、getfloat：获取选项的值
```
import ConfigParser

cf = ConfigParser.ConfigParser(allow_no_value = True)
cf.read('my.cnf')   //创建mysql的副本路径
## ['my.cnf']
cf.sections()
## ['client','mysqld']
cf.has_section('client')
## True
cf.options('client')
## ['port','user','password','host']
cf.has_option('client','user')
## True
cf.get('client','host')
## '127.0.0.1'
cf.getint('client','port')
## 3306
```

ConfigParse中修改配置项相关的方法有：
* remove_section:删除一个章节
* add_section:添加一个章节
* remote_option:删除一个选项
* set:添加一个选项
* write：将ConfigParse对象中的数据保存到文件中
```
# -*- coding: utf-8 -*-
import os
import ConfigParser


cf = ConfigParser.ConfigParser(allow_no_value = True)
cf.read('my.cnf')   ## 读取mysql的副本路径缓存进cf
## ['my.cnf']
cf.sections()
## ['client','mysqld']
cf.has_section('client')
## True
cf.options('client')
## ['port','user','password','host']
cf.has_option('client','user')
## True
cf.get('client','host')
## '127.0.0.1'
cf.getint('client','port')
## 3306

os.system("pause")
print("write")
cf.remove_section('client')
## 
cf.add_section('mysql')
## 增加mysql章节
cf.set('mysql','host','127.0.0.1')
## 在mysql章节下增加hos t = 127.0.0.1
cf.set('mysql','port','3306')
## 在mysql章节下增加port = 3306
cf.write(open('my_copy.cnf','w'))
## 将cf写到my_copy.cnf副本
```


## 3.3使用argparse解析命令行参数

argparse是标准库中用来解析命令行参数的模块；argparse能根据程序中的定义从<font color=#FF1493>sys.argv</font>中解析出这些参数，并自动生成帮助和使用信息

### 3.3.1 ArgumentParse解析器
```
# 使用argparse解析命令行参数前，必须先创建一个解析器
import argparse
parser = argparse.ArgumentParser() 
```
ArgumentParser类的初始化函数有多个参数，其中比较常用的是description,用以程序描述信息。

为应用程序添加参数选项需要用ArgumentParser对象的add_argument方法

```add_argument(name or flags...[,action][,nargs][,const][,default][,type][,choices][,required][,help][,metavar][,dest])```

* name/flags:参数的名字
* action:遇到参数时的动作{store,store_const,store_ture/store_false,append,append_const,version }
* nargs:参数的个数,可以是巨头的数字,或者是“+”号与“*”号。其中,“*”号表示0或多个参数,“+”表示1或多个参数
* const action和nargs:
* default:
* type:参数的类型
* choices:参数允许的值
* required:可选参数是否可以省略
* help:参数的帮助信息
* metavar:在usage说明中的参数名称
* dest:解析后的参数名称

解析参数用ArgumentParser对象的parse_args方法，该方法返回一个Namespace对象。获取对象后，参数值通过属性的方式进行访问


```
from __future__ import print_function
import argparse


def _argparse():
    parser = argparse.ArgumentParser(description="This is description")  #创建ArgumentParser解析器
    #以add_argument函数添加选项，通过parser.server 获取 --host选项的值
    parser.add_argument('--host', action = 'store', dest = 'server', default = "localhost", help= 'connect to host')
    # 通过parse.boolean_switch 获取-t选项的值
    parser.add_argument('-t',action ='store_true', default = False, dest = 'boolean_switch', help = 'Set a switch to true')
    return parser.parse_args()


def main():
    parser= _argparse()
    print(parser)
    print('host = ', parser.server)
    print('boolean_switch=', parser.boolean_switch)


if __name__ =='__main__':
    main()
```
### 3.3.2模仿MySQL客户端的命令参数

```
from __future__ import print_function
import argparse


def _argparse():
    parser = argparse.ArgumentParser(description='A python-mysql client')
    parser.add_argument('--host',action='store',dest='host',required=True,help='connect to host')   #parser.host
    parser.add_argument('-u','--user',action='store',dest='user',required=True,help='user for login')   #parser.user
    parser.add_argument('-p','--password',action='store',dest='password',required=True,help='passwoed to use when connecting to server')    #parser.password
    parser.add_argument('-P','--Port',action='store',dest='port',default=3306,type=int,required=True,help='port number to use for connection or 3306 for default')    #parser.port
    parser.add_argument('-v','--version',action='version',version='%(prog)s 0.1')#parser.version
    return parser.parse_args()#返回add_argument参数的dest中解析出来的值


def main():
    parser = _argparse()
    conn_args = dict(host=parser.host,user=parser.user,password=parser.password,port=parser.port)
    print(conn_args)


if __name__=='__main__':
    main()
```
>关联3.5节click、prompt_toolkit

## 3.4使用logging记录日志
```import logging```

### 3.4.1 日志的作用
>诊断:记录与应用程序操作相关的日志

>审计:为商业分析而记录的日志

### 3.4.2 python 中的logging模块
<center>表3-1 logging 模块日志级别</center>
<table>
    <tr>
        <th width=15%, bgcolor=yellow >日志级别</th>
        <th width=15%, bgcolor=yellow>权重</th>
        <th width=50%, bgcolor=yellow>含义</th>
    </tr>
    <tr>
        <td>CRITICAL</td>    
        <td>50</td>
        <td>严重错误，软件不能继续运行。</td>
    </tr>
    <tr>
        <td>ERROR</td>
        <td>40</td>
        <td>发生严重错误，需要马上处理</td>
    </tr>
    <tr>
        <td>WARNING</td>
        <td>30</td>
        <td>应用程序可以容忍这些信息,但是此时应用程序还是正常运行的，不过它应该被检查及修复，否则将在不久的将来发生问题</td>
    </tr>
    <tr>
        <td>INFO</td>
        <td>20</td>
        <td>证明事情按预期工作，突出强调应用程序的运行过程</td>
    </tr>
    <tr>
        <td>DEBUG</td>
        <td>10</td>
        <td>详细信息，只有开发人员调试程序时才需要关注的事情</td>
    </tr>
</table>

### 3.4.3配置日至格式 

在使用日至之前，一些简单的配置
```
#!/usr/local/bin/python
# -*- coding:utf-8 -*-
import logging

logging.basicConfig(filename='app.log',level=logging.INFO)

logging.debug('debug message')
logging.info('info message')
logging.warn('warn message')
logging.error('error message')
logging.critical('critical message')
```
上述程序会在当前目录产生一个app.log文件。该文件中存在INFO及INFO以上级别的日志记录。
它通过basicConfig方法对日志进行了简单的配置，也可以进行更加复杂的日至配置;logging模块中的几个概念，
> Logger:日志记录器,是应用程序中能直接使用的接口
> Handler:日至处理器,用以表明将日志保存到什么地方以及保存多久
> Fornatter:格式化,，用以配置日志的输出格式

一个日志处理器使用一个日志处理器，一个日志处理器使用一个日志格式化。

```
#!/usr/local/bin/python
#-*- coding:utf-8 -*-
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s:%(levelname)s:%(message)s',
    filename="app.log"
)
logging.debug('debug message')
logging.info('info message')
logging.warn('warn message')
logging.error('error message')
logging.critical('critical message')
```

一个典型的日志配置文件:
```
[loggers]
keys=root//一个名为root的logger

[handlers]
keys=logfile//一个名为logfile的handler

[formatters]
keys=generic//一个名为generic的formatter

[logger_root]
handlers=logfile

[handler_logfile]
class=handlers.TimedRotatingFileHandler
args=('app.log','midnight',1,10)
level=DEBUG
formatter=generic

[formatter_generic]
format=%(asctime)s%(levelname)-5.5s [%(name)s:%(lineno)s]%(message)s
```