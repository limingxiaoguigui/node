### 练习1 （最简单的 express 应用）
##### 安装 express
```
mkdir lesson1 && cd lesson1
# 这里没有从官方 npm 安装，而是使用了大淘宝的 npm 镜像
npm install express --registry=https://registry.npm.taobao.org

npm list  查看是否安装成功
```
##### 新建app.js文件
```
var express = require('express');
var app = express();
app.get('/', function (req, res) {
  res.send('Hello World');
});
app.listen(3000, function () {
  console.log('app is listening at port 3000');
});
```
查看结果
```
node app.js
http://localhost:3000/
```
```
Hello World
```
### 练习2（使用外部模块）
##### 初始化
```
mkdir lesson2 && cd lesson2
npm  init
npm install express utility --save
```
##### 新建app.js文件
```
// 引入依赖
var express = require('express');
var utility = require('utility');
// 建立express实列
var app = express();

app.get('/', function (req, res) {
  // 地址栏参数
  var q = req.query.q;
  var md5Value = utility.md5(q);
  res.send(md5Value)
});
app.listen(3000, function (req, res) {
  console.log('app is running');
})
// http://localhost:3000/?q=123456

```
查看结果
```
node app.js
 http://localhost:3000/?q=123456
```
```
e10adc3949ba59abbe56e057f20f883e
```
### 练习3（使用 superagent 与 cheerio 完成简单爬虫）
##### 初始化
```
mkdir lesson3 && cd lesson3
npm  init
npm install superagent cheerio  express   --save
```
##### 新建app.js文件
```
var express = require("express");
var cheerio = require("cheerio");
var superagent = require("superagent");

var app = express();

app.get("/", function (req, res, next) {
  superagent.get("https://cnodejs.org/").end(function (err, sres) {
    if (err) {
      return next(err);
    }
    var $ = cheerio.load(sres.text);
    var items = [];
    $("#topic_list .topic_title").each(function (idx, element) {
      var $element = $(element);
      items.push({
        title: $element.attr("title"),
        href: $element.attr("href"),
      });
    });

    res.send(items);
  });
});

app.listen(3000, function () {
  console.log("app is listening at port 3000");
});


```
查看结果
```
node app.js
```
```
[
  {
    title: '',
    href: 'https://cnodejs.org/topic/5ec25667a87fc8583363cfb0',
    comment1: ''
  },
  {
    title: '置顶\n\n\n\n        Node.js 开发者调查问卷 [报告已出炉]',
    href: 'https://cnodejs.org/topic/5e4fa8531225c9423dcda9d8',
    comment1: '已填，这个调查有什么现实意义么？比如产生什么价值，除上面三条外。'
 }]
```

### 练习4（使用 eventproxy 控制并发）
##### 初始化
```
mkdir lesson4 && cd lesson4
npm  init
npm install superagent cheerio  eventproxy   --save
```
##### 新建app.js文件
```
var eventproxy = require('eventproxy');
var superagent = require('superagent');
var cheerio = require('cheerio');
var url = require('url');

var cnodeUrl = 'https://cnodejs.org/';

superagent.get(cnodeUrl)
  .end(function (err, res) {
    if (err) {
      return console.error(err);
    }
    var topicUrls = [];
    var $ = cheerio.load(res.text);
    $('#topic_list .topic_title').each(function (idx, element) {
      var $element = $(element);
      var href = url.resolve(cnodeUrl, $element.attr('href'));
      topicUrls.push(href);
    });

    var ep = new eventproxy();

    ep.after('topic_html', topicUrls.length, function (topics) {
      topics = topics.map(function (topicPair) {
        var topicUrl = topicPair[0];
        var topicHtml = topicPair[1];
        var $ = cheerio.load(topicHtml);
        return ({
          title: $('.topic_full_title').text().trim(),
          href: topicUrl,
          comment1: $('.reply_content').eq(0).text().trim(),
        });
      });

      console.log('final:');
      console.log(topics);
    });

    topicUrls.forEach(function (topicUrl) {
      superagent.get(topicUrl)
        .end(function (err, res) {
          console.log('fetch ' + topicUrl + ' successful');
          ep.emit('topic_html', [topicUrl, res.text]);
        });
    });
  });

```
查看结果
```
node app.js
```
```
fetch https://cnodejs.org/topic/5ecccff0a87fc8583363e242 successful
fetch https://cnodejs.org/topic/5ebba355e785ec40b04fc1de successful
final:
[
  {
    title: '',
    href: 'https://cnodejs.org/topic/5ec25667a87fc8583363cfb0',
    comment1: ''
  },
  {
    title: '置顶\n\n\n\n        Node.js 开发者调查问卷 [报告已出炉]',
    href: 'https://cnodejs.org/topic/5e4fa8531225c9423dcda9d8',
    comment1: '已填，这个调查有什么现实意义么？比如产生什么价值，除上面三条外。'
 }]
```
### 练习5（使用 async 控制并发）
##### 初始化
```
mkdir lesson5 && cd lesson5
npm  init
npm install async   --save
```
##### 新建app.js文件
```
var express = require("express");
var cheerio = require("cheerio");
var superagent = require("superagent");

var app = express();

app.get("/", function (req, res, next) {
  superagent.get("https://cnodejs.org/").end(function (err, sres) {
    if (err) {
      return next(err);
    }
    var $ = cheerio.load(sres.text);
    var items = [];
    $("#topic_list .topic_title").each(function (idx, element) {
      var $element = $(element);
      items.push({
        title: $element.attr("title"),
        href: $element.attr("href"),
      });
    });

    res.send(items);
  });
});

app.listen(3000, function () {
  console.log("app is listening at port 3000");
});

```
查看结果
```
node app.js
```
```
现在的并发数是 1 ，正在抓取的是 http://datasource_0 ，耗时1293毫秒
现在的并发数是 2 ，正在抓取的是 http://datasource_1 ，耗时451毫秒
现在的并发数是 3 ，正在抓取的是 http://datasource_2 ，耗时155毫秒
final:
[
  'http://datasource_0 html content',
  'http://datasource_1 html content',
  'http://datasource_2 html content'
]
```
### 练习6（测试用例：mocha，should，istanbul）
##### 初始化
```
mkdir lesson6 && cd lesson6
npm  init
npm install should istanbul  mocha
```
##### 新建main.js
```
var fibonacci = function (n) {
  if (typeof n !== 'number') {
    throw new Error('n should be a Number');
  }
  if (n < 0) {
    throw new Error('n should >= 0');
  }
  if (n > 10) {
    throw new Error('n should <= 10');
  }
  if (n === 0) {
    return 0;
  }
  if (n === 1) {
    return 1;
  }
  return fibonacci(n - 1) + fibonacci(n - 2);
}
exports.fibonacci = fibonacci;

if (require.main === module) {
  var n = Number(process.argv[2]);
  console.log('fibonacci(' + n + ') is', fibonacci(n));
}


```
##### 新建test/main.test.js
```
var main = require('../main');
var should = require('should');

describe('test/main.test.js', function () {
  it('should equal 0 when n === 0', function () {
    main.fibonacci(0).should.equal(0);
  });

  it('should equal 1 when n === 1', function () {
    main.fibonacci(1).should.equal(1);
  });

  it('should equal 55 when n === 10', function () {
    main.fibonacci(10).should.equal(55);
  });

  it('should throw when n > 10', function () {
    (function () {
      main.fibonacci(11);
    }).should.throw('n should <= 10');
  });

  it('should throw when n < 0', function () {
    (function () {
      main.fibonacci(-1);
    }).should.throw('n should >= 0');
  });

  it('should throw when n isnt Number', function () {
    (function () {
      main.fibonacci('呵呵');
    }).should.throw('n should be a Number');
  });
});

```
##### 修改package文件
```
[
 "scripts": {
    "test-mocha": "mocha",
    "test-cov": "istanbul cover node_modules/mocha/bin/_mocha --report html --report text -x \"**.spec.js\""
]

```
##### 测试
```
npm run  test-cov
```
##### 结果
```
λ npm  run test-cov

> lesson6@1.0.0 test-cov D:\NodeCode\lesson6
> istanbul cover node_modules/mocha/bin/_mocha --report html --report text -x "**.spec.js"



  test/main.test.js
    √ should equal 0 when n === 0
    √ should equal 1 when n === 1
    √ should equal 55 when n === 10
    √ should throw when n > 10
    √ should throw when n < 0
    √ should throw when n isnt Number


  6 passing (18ms)

=============================================================================
Writing coverage object [D:\NodeCode\lesson6\coverage\coverage.json]
Writing coverage reports at [D:\NodeCode\lesson6\coverage]
=============================================================================
----------|----------|----------|----------|----------|----------------|
File      |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
----------|----------|----------|----------|----------|----------------|
 lesson6\ |     87.5 |    91.67 |      100 |     87.5 |                |
  main.js |     87.5 |    91.67 |      100 |     87.5 |          22,23 |
----------|----------|----------|----------|----------|----------------|
All files |     87.5 |    91.67 |      100 |     87.5 |                |
----------|----------|----------|----------|----------|----------------|


=============================== Coverage summary ===============================
Statements   : 87.5% ( 14/16 )
Branches     : 91.67% ( 11/12 )
Functions    : 100% ( 1/1 )
Lines        : 87.5% ( 14/16 )
===========================================

```
### 练习7（浏览器端测试：mocha，chai，phantomjs）
##### 初始化
```
mkdir lesson7 && cd lesson7
mkdir vendor
cd vendor
npm  init
npm install  mocha  mocha-phantomjs
mocha init .
```
##### 修改packge.json文件
```
[
"scripts": {
  "test": "mocha-phantomjs index.html --ssl-protocol=any --ignore-ssl-errors=true"
},
]

```
##### 新建tests.js文件
```
var should = chai.should();
describe('simple test', function () {
  it('should equal 0 when n === 0', function () {
    window.fibonacci(0).should.equal(0);
  });
});

```
##### 修改 index.html文件
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Mocha</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="mocha.css" />
  </head>
  <body>
    <div id="mocha"></div>
    <script src="mocha.js"></script>
    <script
      src="https://www.chaijs.com/chai.js"
      type="text/javascript"
    ></script>
    <script>
      var fibonacci = function (n) {
        if (n === 0) {
          return 0;
        }
        if (n === 1) {
          return 1;
        }
        return fibonacci(n - 1) + fibonacci(n - 2);
      };
    </script>

    <script>
      mocha.setup("bdd");
    </script>
    <script src="tests.js"></script>
    <script>
      if (
        window.initMochaPhantomJS &&
        window.location.search.indexOf("skip") === -1
      ) {
        initMochaPhantomJS();
      }
      mocha.ui("bdd");
      expect = chai.expect;

      mocha.run();
    </script>
  </body>
</html>
```
##### 测试结果：
```
λ npm  test

> vendor@1.0.0 test D:\NodeCode\lesson7\vendor
> mocha-phantomjs index.html --ssl-protocol=any --ignore-ssl-errors=true

DeprecationWarning: "useColors()" is DEPRECATED, please use "color()" instead.


  simple test
    ✓ should equal 0 when n === 0
  1 passing (3ms)
```
### 练习8（测试用例：supertest）
##### 初始化
```
mkdir lesson8 && cd lesson8
npm  init
npm  install mocha should supertest express
```
##### 新建app.js
```
var express = require("express");
var fibonacci = function (n) {
  if (typeof n !== "number" || isNaN(n)) {
    throw new Error("n should be a Number");
  }
  if (n < 0) {
    throw new Error("n should >= 0");
  }
  if (n > 10) {
    throw new Error("n should <= 10");
  }
  if (n === 0) {
    return 0;
  }
  if (n === 1) {
    return 1;
  }
  return fibonacci(n - 1) + fibonacci(n - 2);
};
var app = express();
app.get("/fib", function (req, res) {
  var n = Number(req.query.n);
  try {
    // 为何使用 String 做类型转换，是因为如果你直接给个数字给 res.send 的话，
    // 它会当成是你给了它一个 http 状态码，所以我们明确给 String
    res.send(String(fibonacci(n)));
  } catch (e) {
    res.status(500).send(e.message);
  }
});
module.exports = app;

app.listen(3000, function () {
  console.log("app is listening at port 3000");
});
```
##### 浏览器结果
```
http://localhost:3000/fib?n=10
55
```

##### 新建test/app.test.js
```
var app = require("../app");

var supertest = require("supertest");

var request = supertest(app);

var should = require("should");

describe("test/app.test.js", function (done) {
  it("should return 55 when n is 10", function (done) {
    request
      .get("/fib")
      .query({
        n: 10,
      })
      .end(function (err, res) {
        res.text.should.equal("55");
        done(err);
      });
  });
  var testFib = function (n, statusCode, expect, done) {
    request
      .get("/fib")
      .query({
        n: n,
      })
      .expect(statusCode)
      .end(function (err, res) {
        res.text.should.equal(expect);
        done(err);
      });
  };
  it("should return 0 when n === 0", function (done) {
    testFib(0, 200, "0", done);
  });
  it("should equal 1 when n === 1", function (done) {
    testFib(1, 200, "1", done);
  });
  it("should equal 55 when n === 10", function (done) {
    testFib(10, 200, "55", done);
  });
  it("should throw when n > 10", function (done) {
    testFib(11, 500, "n should <= 10", done);
  });
  it("should throw when n < 0", function (done) {
    testFib(-1, 500, "n should >= 0", done);
  });
  it("should throw when n isnt Number", function (done) {
    testFib("goods", 500, "n should be a Number", done);
  });
  it("should status 500 when error", function (done) {
    request
      .get("/fib")
      .query({
        n: 100,
      })
      .expect(500)
      .end(function (err, res) {
        done(err);
      });
  });
});
```
##### 安装nodemon
```
npm i -g nodemon #自动检测 node.js 代码
nodemon app.js
```
##### 修改package.json文件
```
{
    "scripts": {
    "test": "mocha"
  },
}
```
##### 测试结果
```
λ npm run test

> lesson8@1.0.0 test D:\NodeCode\lesson8
> mocha

app is listening at port 3000


  test/app.test.js
    √ should return 55 when n is 10
    √ should return 0 when n === 0
    √ should equal 1 when n === 1
    √ should equal 55 when n === 10
    √ should throw when n > 10
    √ should throw when n < 0
    √ should throw when n isnt Number
    √ should status 500 when error


  8 passing (67ms)

```
### 练习9（正则表达式）
##### 校验数字的表达式
<pre>数字：^[0-9]*$
n位的数字：^\d{n}$
至少n位的数字：^\d{n,}$
m-n位的数字：^\d{m,n}$
零和非零开头的数字：^(0|[1-9][0-9]*)$
非零开头的最多带两位小数的数字：^([1-9][0-9]*)+(\.[0-9]{1,2})?$
带1-2位小数的正数或负数：^(\-)?\d+(\.\d{1,2})$
正数、负数、和小数：^(\-|\+)?\d+(\.\d+)?$
有两位小数的正实数：^[0-9]+(\.[0-9]{2})?$
有1~3位小数的正实数：^[0-9]+(\.[0-9]{1,3})?$
非零的正整数：^[1-9]\d*$ 或 ^([1-9][0-9]*){1,3}$ 或 ^\+?[1-9][0-9]*$
非零的负整数：^\-[1-9][]0-9"*$ 或 ^-[1-9]\d*$
非负整数：^\d+$ 或 ^[1-9]\d*|0$
非正整数：^-[1-9]\d*|0$ 或 ^((-\d+)|(0+))$
非负浮点数：^\d+(\.\d+)?$ 或 ^[1-9]\d*\.\d*|0\.\d*[1-9]\d*|0?\.0+|0$
非正浮点数：^((-\d+(\.\d+)?)|(0+(\.0+)?))$ 或 ^(-([1-9]\d*\.\d*|0\.\d*[1-9]\d*))|0?\.0+|0$
正浮点数：^[1-9]\d*\.\d*|0\.\d*[1-9]\d*$ 或 ^(([0-9]+\.[0-9]*[1-9][0-9]*)|([0-9]*[1-9][0-9]*\.[0-9]+)|([0-9]*[1-9][0-9]*))$
负浮点数：^-([1-9]\d*\.\d*|0\.\d*[1-9]\d*)$ 或 ^(-(([0-9]+\.[0-9]*[1-9][0-9]*)|([0-9]*[1-9][0-9]*\.[0-9]+)|([0-9]*[1-9][0-9]*)))$
浮点数：^(-?\d+)(\.\d+)?$ 或 ^-?([1-9]\d*\.\d*|0\.\d*[1-9]\d*|0?\.0+|0)$
</pre>
##### 校验字符的表达式
<pre>
汉字：^[\u4e00-\u9fa5]{0,}$
英文和数字：^[A-Za-z0-9]+$ 或 ^[A-Za-z0-9]{4,40}$
长度为3-20的所有字符：^.{3,20}$
由26个英文字母组成的字符串：^[A-Za-z]+$
由26个大写英文字母组成的字符串：^[A-Z]+$
由26个小写英文字母组成的字符串：^[a-z]+$
由数字和26个英文字母组成的字符串：^[A-Za-z0-9]+$
由数字、26个英文字母或者下划线组成的字符串：^\w+$ 或 ^\w{3,20}$
中文、英文、数字包括下划线：^[\u4E00-\u9FA5A-Za-z0-9_]+$
中文、英文、数字但不包括下划线等符号：^[\u4E00-\u9FA5A-Za-z0-9]+$ 或 ^[\u4E00-\u9FA5A-Za-z0-9]{2,20}$
可以输入含有^%&',;=?$\"等字符：[^%&',;=?$\x22]+
禁止输入含有~的字符：[^~\x22]+
</pre>
##### 特殊需求表达式
<pre>
Email地址：^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$
域名：[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+\.?
InternetURL：[a-zA-z]+://[^\s]* 或 ^http://([\w-]+\.)+[\w-]+(/[\w-./?%&=]*)?$
手机号码：^(13[0-9]|14[5|7]|15[0|1|2|3|4|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$
电话号码("XXX-XXXXXXX"、"XXXX-XXXXXXXX"、"XXX-XXXXXXX"、"XXX-XXXXXXXX"、"XXXXXXX"和"XXXXXXXX)：^(\(\d{3,4}-)|\d{3.4}-)?\d{7,8}$
国内电话号码(0511-4405222、021-87888822)：\d{3}-\d{8}|\d{4}-\d{7}
电话号码正则表达式（支持手机号码，3-4位区号，7-8位直播号码，1－4位分机号）: ((\d{11})|^((\d{7,8})|(\d{4}|\d{3})-(\d{7,8})|(\d{4}|\d{3})-(\d{7,8})-(\d{4}|\d{3}|\d{2}|\d{1})|(\d{7,8})-(\d{4}|\d{3}|\d{2}|\d{1}))$)
身份证号(15位、18位数字)，最后一位是校验位，可能为数字或字符X：(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)
帐号是否合法(字母开头，允许5-16字节，允许字母数字下划线)：^[a-zA-Z][a-zA-Z0-9_]{4,15}$
密码(以字母开头，长度在6~18之间，只能包含字母、数字和下划线)：^[a-zA-Z]\w{5,17}$
强密码(必须包含大小写字母和数字的组合，不能使用特殊字符，长度在 8-10 之间)：^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])[a-zA-Z0-9]{8,10}$
强密码(必须包含大小写字母和数字的组合，可以使用特殊字符，长度在8-10之间)：^(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,10}$
日期格式：^\d{4}-\d{1,2}-\d{1,2}
一年的12个月(01～09和1～12)：^(0?[1-9]|1[0-2])$
一个月的31天(01～09和1～31)：^((0?[1-9])|((1|2)[0-9])|30|31)$
xml文件：^([a-zA-Z]+-?)+[a-zA-Z0-9]+\\.[x|X][m|M][l|L]$
中文字符的正则表达式：[\u4e00-\u9fa5]
双字节字符：[^\x00-\xff] (包括汉字在内，可以用来计算字符串的长度(一个双字节字符长度计2，ASCII字符计1))
空白行的正则表达式：\n\s*\r (可以用来删除空白行)
HTML标记的正则表达式：<(\S*?)[^>]*>.*?|<.*? /> ( 首尾空白字符的正则表达式：^\s*|\s*$或(^\s*)|(\s*$) (可以用来删除行首行尾的空白字符(包括空格、制表符、换页符等等)，非常有用的表达式)
腾讯QQ号：[1-9][0-9]{4,} (腾讯QQ号从10000开始)
中国邮政编码：[1-9]\d{5}(?!\d) (中国邮政编码为6位数字)
IP地址：((?:(?:25[0-5]|2[0-4]\\d|[01]?\\d?\\d)\\.){3}(?:25[0-5]|2[0-4]\\d|[01]?\\d?\\d))
钱的输入格式：
    1. 有四种钱的表示形式我们可以接受:"10000.00" 和 "10,000.00", 和没有 "分" 的 "10000" 和 "10,000"：^[1-9][0-9]*$
    2. 这表示任意一个不以0开头的数字,但是,这也意味着一个字符"0"不通过,所以我们采用下面的形式：^(0|[1-9][0-9]*)$
    3. 一个0或者一个不以0开头的数字.我们还可以允许开头有一个负号：^(0|-?[1-9][0-9]*)$
    4. 这表示一个0或者一个可能为负的开头不为0的数字.让用户以0开头好了.把负号的也去掉,因为钱总不能是负的吧。下面我们要加的是说明可能的小数部分：^[0-9]+(.[0-9]+)?$
    5. 必须说明的是,小数点后面至少应该有1位数,所以"10."是不通过的,但是 "10" 和 "10.2" 是通过的：^[0-9]+(.[0-9]{2})?$
    6. 这样我们规定小数点后面必须有两位,如果你认为太苛刻了,可以这样：^[0-9]+(.[0-9]{1,2})?$
    7. 这样就允许用户只写一位小数.下面我们该考虑数字中的逗号了,我们可以这样：^[0-9]{1,3}(,[0-9]{3})*(.[0-9]{1,2})?$
    8. 1到3个数字,后面跟着任意个 逗号+3个数字,逗号成为可选,而不是必须：^([0-9]+|[0-9]{1,3}(,[0-9]{3})*)(.[0-9]{1,2})?$
    9. 备注：这就是最终结果了,别忘了"+"可以用"*"替代如果你觉得空字符串也可以接受的话(奇怪,为什么?)最后,别忘了在用函数时去掉去掉那个反斜杠,一般的错误都在这里
</pre>
### 练习10（benchmark）
##### 初始化
```
mkdir lesson10 && cd lesson10
npm  init
npm  install benchmark
```
##### 新建main.js
```
var Benchmark = require("benchmark");
var suite = new Benchmark.Suite();

var int1 = function (str) {
  return +str;
};
var int2 = function (str) {
  return parseInt(str, 10);
};

var int3 = function (str) {
  return Number(str);
};
var number = "100";
suite
  .add("+", function () {
    int1(number);
  })
  .add("parseInt", function () {
    int2(number);
  })
  .add("Number", function () {
    int3(number);
  })
  .on("cycle", function (event) {
    console.log(String(event.target));
  })
  .on("complete", function () {
    console.log("Fastest is " + this.filter("fastest").map("name"));
  })
  .run({
    async: true,
  });

```
##### 测试结果
```
λ node main.js
+ x 58,742,061 ops/sec ±10.48% (43 runs sampled)
parseInt x 35,091,517 ops/sec ±6.78% (68 runs sampled)
Number x 45,408,686 ops/sec ±5.06% (67 runs sampled)
Fastest is +
```
### 练习11（作用域、闭包、this）
##### 作用域
```
1. 内部函数可以访问外部函数的变量，外部不能访问内部函数的变量。
2. 如果忘记var，那么变量就被声明为全局变量了。
3. 变量 i 和 value 虽然是在for循环中变量 i 和 value 都有中被定义，但在代码块外仍可以访问 i 和 value。

```
##### 闭包
```
 1. 闭包就是使内部函数可以访问定义在外部函数中的变量。
 2. 闭包的坑
 for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i); // 五个5
  }, 5);
 }
 3. 例子
 var adder = function (x) {
 var base = x;
 return function (n) {
    return n + base;
   };
 };

var add10 = adder(10);
```
##### this
```
1. 函数有所属对象时：指向所属对象
2. 函数没有所属对象：指向全局对象
3. 构造器中的 this：指向新对象
4. apply 和 call 调用以及 bind 绑定：指向绑定的对象

```
