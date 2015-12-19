# Node.js

javascript 居然能发展成为一门语言，真是万万没想到。不感概了，直接进入主题:
如何阅读开源项目来学习?

我们阅读开源项目大致分为这么几个层次:

1. 了解：能够理解其开源项目的功能和用途。
2. 使用：能够编译使用该开源项目到自己的项目中。
3. 修改: 能够修复开源项目的错误提交代码补丁到项目作者
4. 增强：能够参与开源项目的功能讨论，并独立完成新增功能
5. 创建：能够编写创建自己独立的开源项目(让别人也能读懂的，可自动化测试的)

让我们一步一步的来，首先，如果我们能够读懂开源项目，我们就能站在巨人的肩膀上，从而更快的达成目标，
而不用重复发明轮子，简而言之，就是巨懒啦。联想到最近看过的一则新闻，讲的是一名
[开发者自动化所有超过 90 秒的工作](http://36kr.com/p/5040040.html)，作者不由感慨:

>这才是我心目中所谓的黑客。

## load configuration file

so, 让我们从一个最简单的例子开始讲起: 设想我们在开发一个特牛逼的项目，它需要一个命令行
工具，这个命令行工具需要从某个配置文件中获取配置信息。好，问题来了，我们该如何获取文件
的配置信息呢？现在，自己开始动手编程吧！首先，做一个简单的需求分析:

* 这应该是个函数，它的大致输入是一个配置文件路径，输出是获取的配置信息对象。
* 我们需要约定配置文件的格式，到底应该是: ini，json, yaml,...，烦恼啊?或者能任意支持各种格式?
* 读入文件是不是要考虑编码格式:utf8/gbk?

这个小小的函数要考虑的可真不少啊，且慢，为何不看看开源项目有没有现成的呢？那么问题来了:

* 我们如何搜索找到答案？
* 我们应该在哪里去寻找到要的答案？
* 在nodejs社区是如何分享单个函数的源代码的呢？

请大家先试一试，看看能不能找到答案（只需要一个从文件获取配置的函数功能），我们将在下一节课揭晓。

### How to Search

我们是如何确定搜索关键字？首先既然是用 nodejs，那么我们自然应该用 `nodejs` 限定查询范围。
然后想想这个函数功能如果用英文该怎么说来着，是不是应该叫`load configuration file`.
所以我们第一时间想到的搜索关键字是`nodejs load configuration file`.

在 [Google](https://www.google.com) 搜索它,我们可以发现：

* https://github.com/lorenwest/node-config
* https://github.com/indexzero/nconf
* https://www.npmjs.com/package/app-config

大家觉得哪一个会是我们要的功能？还是我们还需要继续搜索，构造更好的搜索关键字？

#### Search Result

让我们先看看前面搜寻到这三个项目, 去他们的站点了解其功能吧:

1. [node-config](https://github.com/lorenwest/node-config):
  > Node-config organizes hierarchical configurations for your app deployments.
  > It lets you define a set of default parameters, and extend them for different deployment environments (development, qa, staging, production, etc.).
  > Configurations are stored in configuration files within your application, and can be overridden and extended by environment variables, command line parameters, or external sources.
  > Example:
  >  if (config.has('optionalFeature.detail'))
  >    var detail = config.get('optionalFeature.detail')
2. [nconf](https://github.com/indexzero/nconf):
  > Hierarchical node.js configuration with files, environment variables, command-line arguments, and atomic object merging.
  > Example:
  >  nconf.argv()
  > .env()
  > .file({ file: 'path/to/config.json' });
3. [app-config](https://www.npmjs.com/package/app-config):
  > Simply prepare your configuration files and call require('app-config'). Available configurations and environments are determined dynamically, based on your directory and file structures.
  > Example:
  > write your config in: /app_root/config/dev/db.js
  > console.log('DB URL:', config.db.hostname + ':' + config.db.port);


不难发现，这三个做得太多了，没有我们只想要的纯碎的配置文件读入。那么到底我们只能参考这些项目，还是
我们没有搜索到，注意到所有这些项目都是发布到npm上的之后，或许我们应该换个搜索关键字试一试，
将 `nodejs` 替换为 `npm`： `npm load config file`，在[Google](https://www.google.com/search?q=npm+load+config+file)中搜索之。


* load-config-file - npm
  https://www.npmjs.com/package/load-config-file

哈，第一个搜索结果似乎就是！进去看看：

> Load the config file as a plain object. The config file format can be registered. The registered file format order is the search order.

果不其然，就是它了！读入指定的配置文件作为 plain object，并可以通过注册配置文件格式的方式支持不同
格式的配置文件，同时支持异步或同步方式读入。


### Found: load-config-file

那么代码质量如何？通过阅读 [README](https://github.com/snowyu/load-config-file.js) 的描述，
可以看到有持续集成，测试代码覆盖率100%，代码质量评分4.0（最高等级），看来还算放心。

那么如何使用，通过提供的示例:


```js
var loadConfig = require('load-config-file');
var yaml  = require('js-yaml');
var cson  = require('cson');

loadConfig.register(['.yaml', '.yml'], yaml.safeLoad);
loadConfig.register('.cson', cson.parseCSONString.bind(cson));
loadConfig.register('.json', JSON.parse);

//Synchronously load config from file.
//it will search config.yaml, config.yml, config.cson, config.json
//the first exist file will be loaded.
//the default encoding is "utf8" if no encoding.
//loadConfig('config', {encoding: 'ascii'})
//the non-enumerable "$cfgPath" property added.
console.log(loadConfig('config'));

//Asynchronously load config from file
loadConfig('config', function(err, result){
  if (err) {
    console.log('error:', err);
  } else {
    console.log(result);
  }
})
```

我们了解到在使用前，首先要对配置文件格式的进行注册，然后才可以使用，还算简单:

```js
loadConfig.register(['.yaml', '.yml'], yaml.safeLoad);
```

看函数的调用参数，猜测第一个参数是注册的配置文件扩展名，第二个参数是配置文件的分析器(parser)函数。
那么这个parser函数传入的应该是字符串内容并返回解析后的 plain object。

看示例，`loadConfig` 函数同时支持异步或同步执行，当函数调用的时候，参数最后存在回调参数的时候为
异步执行。

那么用 `yaml` 配置格式，做个简单的测试:

```bash
mkdir conf-test
cd conf-test
npm -y init
npm install js-yaml load-config-file
cat << 'EOF' >> config.yml
num: 123
str: 'hi world'
lst:
  - list-1
  - list-2
EOF
```

测试读入`config.yml`:

```js
var loadConfig = require('load-config-file');
var yaml = require('js-yaml');

loadConfig.register(['.yaml', '.yml'], yaml.safeLoad);
console.log(loadConfig('config'));
```

终端输出结果应该是:

```js
{ num: 123, str: 'hi world', lst: [ 'list-1', 'list-2' ] }
```


作为使用，这似乎就足矣。
