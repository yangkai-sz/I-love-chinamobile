# 1、Python-nmap是什么

# 2、环境搭建
1、安装nmap  <br>
2、安装python-nmap库
  这个安装方法有点不技巧，不能直接在pycharm里import nmap，然后点那个电灯泡来install，这样是不对的，应该在命令行执行pip3 install python-nmap，或者打开pycharm在设置、解释器那里搜索python-nmap来安装。<br>
3、修改nmap.py（安装之后对应的目录里会有这个文件，扫描主要就是这个里面的3个类），
`def __init__(self, nmap_search_path=('nmap', '/usr/bin/nmap', '/usr/local/bin/nmap', '/sw/bin/nmap', '/opt/local/bin/nmap')):`,nmap_search_path查找nmap的路径（从nmap_search_path可以看出在Windows下使用得自己添加路径）。

# 3、python-nmap的三个类的使用介绍

## 3.1 PortScanner()普通扫描
这个就是常规的普通扫描：
```
import nmap
nm = nmap.PortScanner()
#nm.scan('192.168.10.10-100', ports=None, '22,21','-sV')

res1 = nm.scan(hosts='172.25.209.22', ports='3389,145,11', arguments='-sV')
#--直接print（res）也可以的。最好直接res1就好了（1）

#res = nm.command_line()--返回nmap的命令，没啥用
#res2 = nm.scaninfo() #--返回扫描的信息，没啥用
#res = nm.get_nmap_last_output() --可以的，返回的信息比较多（2）
print(res1)
```

ip作为参数传递，函数好好封装一下，以方便调用。
## 3.2 PortScannerAsync()异步扫描，怎么搞都失败
可能是我window的问题，linux懒得测试了。

## 3.3 PortScannerYield()异步扫描、迭代器
最后返回了一个迭代器，代码简单，使用方便。源代码背后用的是多线程的思想。

这篇文章写的不错：https://blog.51cto.com/coosh/2129009
```
import nmap
nm = nmap.PortScannerYield()
for result in nm.scan('172.25.209.22', ports=None, arguments="-sS"):
    print(result)
 ```
 ### 输出结果：
`('172.25.209.22',
{
  'nmap': {
    'command_line': '"C:/Program Files (x86)/Nmap/nmap.exe" -oX - -sS 172.25.209.22',
    'scaninfo': {
      'tcp': {
        'method': 'syn',
        'services': '1,3-4,.....省略很多内容...65000,65129,65389'
      }
    },
    'scanstats': {
      'timestr': 'Tue Feb 25 20:46:21 2020',
      'elapsed': '0.45',
      'uphosts': '1',
      'downhosts': '0',
      'totalhosts': '1'
    }
  },
  'scan': {
    '172.25.209.22': {
      'hostnames': [
        {
          'name': 'bogon',
          'type': 'PTR'
        }
      ],
      'addresses': {
        'ipv4': '172.25.209.22'
      },
      'vendor': {
        
      },
      'status': {
        'state': 'up',
        'reason': 'localhost-response'
      },
      'tcp': {
        135: {
          'state': 'open',
          'reason': 'syn-ack',
          'name': 'msrpc',
          'product': '',
          'version': '',
          'extrainfo': '',
          'conf': '3',
          'cpe': ''
        },
        139: {
          'state': 'open',
          'reason': 'syn-ack',
          'name': 'netbios-ssn',
          'product': '',
          'version': '',
          'extrainfo': '',
          'conf': '3',
          'cpe': ''
        },
        445: {
          'state': 'open',
          'reason': 'syn-ack',
          'name': 'microsoft-ds',
          'product': '',
          'version': '',
          'extrainfo': '',
          'conf': '3',
          'cpe': ''
        },
        902: {
          'state': 'open',
          'reason': 'syn-ack',
          'name': 'iss-realsecure',
          'product': '',
          'version': '',
          'extrainfo': '',
          'conf': '3',
          'cpe': ''
        },
        912: {
          'state': 'open',
          'reason': 'syn-ack',
          'name': 'apex-mesh',
          'product': '',
          'version': '',
          'extrainfo': '',
          'conf': '3',
          'cpe': ''
        },
        3389: {
          'state': 'open',
          'reason': 'syn-ack',
          'name': 'ms-wbt-server',
          'product': '',
          'version': '',
          'extrainfo': '',
          'conf': '3',
          'cpe': ''
        },
        6000: {
          'state': 'open',
          'reason': 'syn-ack',
          'name': 'X11',
          'product': '',
          'version': '',
          'extrainfo': '',
          'conf': '3',
          'cpe': ''
        },
        8200: {
          'state': 'open',
          'reason': 'syn-ack',
          'name': 'trivnet1',
          'product': '',
          'version': '',
          'extrainfo': '',
          'conf': '3',
          'cpe': ''
        }
      }
    }
  }
})`
### 使用nmap单独扫描的结果：
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-02-25 20:51 China Standard Time

NSE: Loaded 151 scripts for scanning.

NSE: Script Pre-scanning.

Initiating NSE at 20:51

Completed NSE at 20:51, 0.00s elapsed

Initiating NSE at 20:51

Completed NSE at 20:51, 0.00s elapsed

Initiating NSE at 20:51

Completed NSE at 20:51, 0.00s elapsed

Initiating Parallel DNS resolution of 1 host. at 20:51

Completed Parallel DNS resolution of 1 host. at 20:51, 0.00s elapsed

Initiating SYN Stealth Scan at 20:51

Scanning bogon (172.25.209.22) [1000 ports]

Discovered open port 139/tcp on 172.25.209.22

Discovered open port 445/tcp on 172.25.209.22

Discovered open port 135/tcp on 172.25.209.22

Discovered open port 3389/tcp on 172.25.209.22

Discovered open port 912/tcp on 172.25.209.22

Discovered open port 8200/tcp on 172.25.209.22

Discovered open port 902/tcp on 172.25.209.22

Discovered open port 6000/tcp on 172.25.209.22

Completed SYN Stealth Scan at 20:51, 0.06s elapsed (1000 total ports)

Initiating Service scan at 20:51

Scanning 8 services on bogon (172.25.209.22)

Completed Service scan at 20:53, 122.75s elapsed (8 services on 1 host)
```
# 4、关于效率问题
效率、准确性，这两个因素很关键，但是我还没对比分析。理论上来说Python这种更快。

## 4.1 Python绝技中使用optparse、socket也可以写扫描器

## 4.2 scapy也可以，但没试过

## 4.3 分布式扫描问题
在谷歌上也搜了半天，没找到分布式的方案，最low的方案，就是自己用部署多个节点，利用round-robin算法，人工实现分布式、负载均衡。毕竟每个节点上，还有对应的方法可以获取当前的扫描状态。
### 思路：总体扫描任务————5个扫描节点，部署上述程序，扫描中心根据扫描任务、节点执行情况，使用round-robin算法来实现
