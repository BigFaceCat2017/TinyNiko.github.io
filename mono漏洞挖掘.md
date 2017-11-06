# Mono 框架漏洞挖掘
# 信息收集
secure@microsoft.com Mono相关问题发送给这个邮箱
http://www.mono-project.com/docs/ 这个包含了很多信息。鉴于.net是ms的，所以先尝试在linux 进行测试。
https://github.com/mono/mono 这个也是比较有用的
# Mono漏洞
CVE-2009-0689  string-to-double 解析漏洞，可以导致ace,影响4.2.0之前版本
```c#
using System;
class Test
{
    static void Main()
    {
        string input = "1." + new string('1', 294912);
        Double.Parse(input);
    }
}
``` 

TLS 漏洞CVE-2015-2318, CVE-2015-2319, CVE-2015-2320
这块漏洞不是我擅长的范围，暂时忽略

CVE-2011-0989 貌似是修改了内部代码之后，造成的崩溃，版本有点老(主要是2.x)，看一下就好
CVE-2011-0990 race in Array.Copy 也是2.x版本
CVE-2010-1526 Libgdiplus Integer Overflow Vulnerabilities
CVE-2005-0509 Mono ASP.NET Unicode Conversion Cross-Site Scripting
CVE-2006-2658 XSP/mod_mono directory traversal
CVE-2008-3906 Mono System.Web Header Injection Attack
看了几个其他的cve，也有一些逻辑漏洞
时间上都比较久远 ，不知道是不是ms接手了mono后都不在官方的文档里更新了 ,上面列举了一些漏洞，从这些漏洞中，我觉得可以从mono跟web相关的地方进行分析，找一些跟web安全相关的漏洞，这个可能需要自己搭建一个服务器之类的，其他的就是mono的组件了，这些组件里包含的漏洞类型也是挺多的，可以从黑白盒方向入手


## 漏洞分析




## 挖掘


