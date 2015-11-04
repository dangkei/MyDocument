# mysql

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]
[![Node.js Version][node-version-image]][node-version-url]
[![Linux Build][travis-image]][travis-url]
[![Windows Build][appveyor-image]][appveyor-url]
[![Test Coverage][coveralls-image]][coveralls-url]

## Table of Contents

- [Install](#install)
- [Introduction](#introduction)
- [Contributors](#contributors)
- [Sponsors](#sponsors)
- [Community](#community)
- [Establishing connections](#establishing-connections)
- [Connection options](#connection-options)
- [SSL options](#ssl-options)
- [Terminating connections](#terminating-connections)
- [Pooling connections](#pooling-connections)
- [Pool options](#pool-options)
- [Pool events](#pool-events)
- [Closing all the connections in a pool](#closing-all-the-connections-in-a-pool)
- [PoolCluster](#poolcluster)
- [PoolCluster Option](#poolcluster-option)
- [Switching users and altering connection state](#switching-users-and-altering-connection-state)
- [Server disconnects](#server-disconnects)
- [Performing queries](#performing-queries)
- [Escaping query values](#escaping-query-values)
- [Escaping query identifiers](#escaping-query-identifiers)
- [Preparing Queries](#preparing-queries)
- [Custom format](#custom-format)
- [Getting the id of an inserted row](#getting-the-id-of-an-inserted-row)
- [Getting the number of affected rows](#getting-the-number-of-affected-rows)
- [Getting the number of changed rows](#getting-the-number-of-changed-rows)
- [Getting the connection ID](#getting-the-connection-id)
- [Executing queries in parallel](#executing-queries-in-parallel)
- [Streaming query rows](#streaming-query-rows)
- [Piping results with Streams2](#piping-results-with-streams2)
- [Multiple statement queries](#multiple-statement-queries)
- [Stored procedures](#stored-procedures)
- [Joins with overlapping column names](#joins-with-overlapping-column-names)
- [Transactions](#transactions)
- [Timeouts](#timeouts)
- [Error handling](#error-handling)
- [Exception Safety](#exception-safety)
- [Type casting](#type-casting)
- [Connection Flags](#connection-flags)
- [Debugging and reporting problems](#debugging-and-reporting-problems)
- [Running tests](#running-tests)
- [Todo](#todo)

## Install

```sh
$ npm install mysql
```

For information about the previous 0.9.x releases, visit the [v0.9 branch][].

Sometimes I may also ask you to install the latest version from Github to check
if a bugfix is working. In this case, please do:

//有时候我也许会需要你们从Github安装最新版本,如果正好有bug正在修改的情况,请这样做:

```sh
$ npm install felixge/node-mysql
```

[v0.9 branch]: https://github.com/felixge/node-mysql/tree/v0.9

## Introduction

This is a node.js driver for mysql. It is written in JavaScript, does not
require compiling, and is 100% MIT licensed.

这个是一个node.js的mysql驱动插件,由Javascript语言编写.不需要编译. 100%遵循MIT协议.

Here is an example on how to use it:

这是一个怎样使用这个驱动的例子.

```js
var mysql      = require('mysql');
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'me',
  password : 'secret',
  database : 'my_db'
});

connection.connect();

connection.query('SELECT 1 + 1 AS solution', function(err, rows, fields) {
  if (err) throw err;

  console.log('The solution is: ', rows[0].solution);
});

connection.end();
```

From this example, you can learn the following:

从这个例子可以看到:

* Every method you invoke on a connection is queued and executed in sequence.
* 每个你调用的方法都排列在一个connection的执行队列中
* Closing the connection is done using `end()` which makes sure all remaining
  queries are executed before sending a quit packet to the mysql server.
* 确认所有剩余的查询语句执行完毕前发送退出包给mysql服务器,关闭connection使用end()

## Contributors
## 贡献人
Thanks goes to the people who have contributed code to this module, see the
[GitHub Contributors page][].

[GitHub Contributors page]: https://github.com/felixge/node-mysql/graphs/contributors

Additionally I'd like to thank the following people:

另外我还非常感谢一下几位:

* [Andrey Hristov][] (Oracle) - for helping me with protocol questions.
* [Ulf Wendel][] (Oracle) - for helping me with protocol questions.

[Ulf Wendel]: http://blog.ulf-wendel.de/
[Andrey Hristov]: http://andrey.hristov.com/

## Sponsors
## 赞助商

The following companies have supported this project financially, allowing me to
spend more time on it (ordered by time of contribution):

以下几个on公司在财务上支持这个项目,这让我又更多的时候花在这个项目里.(排名按贡献时间)

* [Transloadit](http://transloadit.com) (my startup, we do file uploading &
  video encoding as a service, check it out)
* [Joyent](http://www.joyent.com/)
* [pinkbike.com](http://pinkbike.com/)
* [Holiday Extras](http://www.holidayextras.co.uk/) (they are [hiring](http://join.holidayextras.co.uk/))
* [Newscope](http://newscope.com/) (they are [hiring](http://www.newscope.com/stellenangebote))

If you are interested in sponsoring a day or more of my time, please

如果你有兴趣多赞助我几天或者更多时间请联系我.

[get in touch][].

[get in touch]: http://felixge.de/#consulting

## Community
## 沟通

If you'd like to discuss this module, or ask questions about it, please use one
of the following:

如果你喜欢参加这个模块的讨论,或者提出关于它的问题,请到以下网站:

* **Mailing list**: https://groups.google.com/forum/#!forum/node-mysql
* **IRC Channel**: #node.js (on freenode.net, I pay attention to any message
  including the term `mysql`)

## Establishing connections
## 创建连接
The recommended way to establish a connection is this:

推荐使用下面的方法创建connection
```js
var mysql      = require('mysql');
var connection = mysql.createConnection({
  host     : 'example.org',
  user     : 'bob',
  password : 'secret'
});

connection.connect(function(err) {
  if (err) {
    console.error('error connecting: ' + err.stack);
    return;
  }

  console.log('connected as id ' + connection.threadId);
});
```

However, a connection can also be implicitly established by invoking a query:

一个connection也能用调用查询的方法来隐式创建

```js
var mysql      = require('mysql');
var connection = mysql.createConnection(...);

connection.query('SELECT 1', function(err, rows) {
  // connected! (unless `err` is set)
});
```

Depending on how you like to handle your errors, either method may be
appropriate. Any type of connection error (handshake or network) is considered
a fatal error, see the [Error Handling](#error-handling) section for more
information.

这依赖你习惯怎么处理错误. 每种方法都可以,任何类型的网络连接错误(握手或者网络)都被认为是致命错误.
参考错误处理部分了解更新信息.

## Connection options
## 连接选项

When establishing a connection, you can set the following options:

当创建连接时你可以设置以下参数:

* `host`: The hostname of the database you are connecting to. (Default:
  `localhost`)
* `port`: The port number to connect to. (Default: `3306`)
* `localAddress`: The source IP address to use for TCP connection. (Optional)
* `socketPath`: The path to a unix domain socket to connect to. When used `host`
  and `port` are ignored.
* unix domain连接时使用. 使用host和port是忽略这个参数
* `user`: The MySQL user to authenticate as.
* `password`: The password of that MySQL user.
* `database`: Name of the database to use for this connection (Optional).
* `charset`: The charset for the connection. This is called "collation" in the SQL-level
  of MySQL (like `utf8_general_ci`). If a SQL-level charset is specified (like `utf8mb4`)
  then the default collation for that charset is used. (Default: `'UTF8_GENERAL_CI'`)
* 连接字符集,如果连接的Mysql的SQL-level中指定了"collation"时需要设置.(缺省`'UTF8_GENERAL_CI'`)
* `timezone`: The timezone used to store local dates. (Default: `'local'`)
* `connectTimeout`: The milliseconds before a timeout occurs during the initial connection
  to the MySQL server. (Default: `10000`)
* `stringifyObjects`: Stringify objects instead of converting to values. See
issue [#501](https://github.com/felixge/node-mysql/issues/501). (Default: `'false'`)
* Stringify对象替代转换,(缺省false)
* `insecureAuth`: Allow connecting to MySQL instances that ask for the old
  (insecure) authentication method. (Default: `false`)
* `typeCast`: Determines if column values should be converted to native
   JavaScript types. (Default: `true`)
* 确定是否将列值转换为原生Javascript类型. (缺省 true)
* `queryFormat`: A custom query format function. See [Custom format](#custom-format).
*  自定义查询格式化方法,见[Custom format]
* `supportBigNumbers`: When dealing with big numbers (BIGINT and DECIMAL columns) in the database,
  you should enable this option (Default: `false`).
* 当需要处理长数字类型时(数据库列为BIGINT和DECIMAL)你需要启动这个选项
* `bigNumberStrings`: Enabling both `supportBigNumbers` and `bigNumberStrings` forces big numbers
  (BIGINT and DECIMAL columns) to be always returned as JavaScript String objects (Default: `false`).
  Enabling `supportBigNumbers` but leaving `bigNumberStrings` disabled will return big numbers as String
  objects only when they cannot be accurately represented with [JavaScript Number objects] (http://ecma262-5.com/ELS5_HTML.htm#Section_8.5)
  (which happens when they exceed the [-2^53, +2^53] range), otherwise they will be returned as
  Number objects. This option is ignored if `supportBigNumbers` is disabled.
* 如果`supportBigNumbers`设置为true了. `bigNumberStrings`,同时为true, 则当Javascript number object不能准确描述长数字类型时
  会使用javascript String object来处理. 当`supportBigNumbers` 为false是这个参数忽略没用.

* `dateStrings`: Force date types (TIMESTAMP, DATETIME, DATE) to be returned as strings rather then
   inflated into JavaScript Date objects. (Default: `false`)
* 强制时间类型转换为字符串而不是Javascript的日期对象. 

* `debug`: Prints protocol details to stdout. (Default: `false`)
* 是否输出协议的详细信息.
* `trace`: Generates stack traces on `Error` to include call site of library
   entrance ("long stack traces"). Slight performance penalty for most calls.
   (Default: `true`)
* onerror的时候生成错误跟踪数据,对于大多数调用方法会有轻微的性能损耗.
*    
* `multipleStatements`: Allow multiple mysql statements per query. Be careful
  with this, it could increase the scope of SQL injection attacks. (Default: `false`)
* 允许一次查询使用多条mysql语句. 小心使用,因为这样会提高注入攻击的可能.  

* `flags`: List of connection flags to use other than the default ones. It is
  also possible to blacklist default ones. For more information, check
  [Connection Flags](#connection-flags).
* 缺省连接标志值意外的可选值,也可以用来设置黑名单. 更多信息查看[Connection Flags].

* `ssl`: object with ssl parameters or a string containing name of ssl profile. See [SSL options](#ssl-options).
* `ssl`的参数对象, 描述sll属性的字符串. 参见[SSL options].

In addition to passing these options as an object, you can also use a url
string. For example:
* 另外这些选项除了可以使用Javascript对象描述,也可以使用练级字符串来描述.

```js
var connection = mysql.createConnection('mysql://user:pass@host/db?debug=true&charset=BIG5_CHINESE_CI&timezone=-0700');
```

Note: The query values are first attempted to be parsed as JSON, and if that
fails assumed to be plaintext strings.

! 注意: 查询选项首先会试图解析成JSON,如果失败才会尝试使用字符串解析.

### SSL options

The `ssl` option in the connection options takes a string or an object. When given a string,
it uses one of the predefined SSL profiles included. The following profiles are included:

 `ssl` 选项连接的参数是一个字符串或者对象. 当使用字符串的时候,他是包含在一个预定义的ssl属性文件里的.
类似下面内容是一个连接到`"Amazon RDS"`服务器的安全验证文件的内容.

* `"Amazon RDS"`: this profile is for connecting to an Amazon RDS server and contains the
  certificates from https://rds.amazonaws.com/doc/rds-ssl-ca-cert.pem and
  https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem

When connecting to other servers, you will need to provide an object of options, in the
same format as [crypto.createCredentials](http://nodejs.org/api/crypto.html#crypto_crypto_createcredentials_details).
Please note the arguments expect a string of the certificate, not a file name to the
certificate. Here is a simple example:

当连接到其它服务器你需要提供一个这样格式的文件.http://nodejs.org/api/crypto.html#crypto_crypto_createcredentials_details
! 注意:下面的参数是读取的验证文件字符串的内容. 而不是文件的名字.

```js
var connection = mysql.createConnection({
  host : 'localhost',
  ssl  : {
    ca : fs.readFileSync(__dirname + '/mysql-ca.crt')
  }
});
```

You can also connect to a MySQL server without properly providing the appropriate
CA to trust. _You should not do this_.

```js
var connection = mysql.createConnection({
  host : 'localhost',
  ssl  : {
    // DO NOT DO THIS
    // set up your ca correctly to trust the connection
    rejectUnauthorized: false
  }
});
```

## Terminating connections
## 断开连接

There are two ways to end a connection. Terminating a connection gracefully is
done by calling the `end()` method:

有两种方法可以断开连接. 优雅的方法是直接调用`end()`,

```js
connection.end(function(err) {
  // The connection is terminated now
});
```

This will make sure all previously enqueued queries are still before sending a
`COM_QUIT` packet to the MySQL server. If a fatal error occurs before the
`COM_QUIT` packet can be sent, an `err` argument will be provided to the
callback, but the connection will be terminated regardless of that.

这种方法会确认所有队列中的查询已经发送`COM_QUIT`包给MySQL服务器. 如果发生致命错误
还会返回一个包含`err`参数的回调函数. 最后connection断开.

An alternative way to end the connection is to call the `destroy()` method.
This will cause an immediate termination of the underlying socket.
Additionally `destroy()` guarantees that no more events or callbacks will be
triggered for the connection.

另一种方法调用`destroy()`, 这样会导致立即中断底层的socket连接. 另外不会触发任何事件和回调函数.

```js
connection.destroy();
```

Unlike `end()` the `destroy()` method does not take a callback argument.
和`end()`不同`destroy()`没有回调函数参数.

## Pooling connections
## 连接池

Use pool directly.

直接使用连接池

```js
var mysql = require('mysql');
var pool  = mysql.createPool({
  connectionLimit : 10,
  host            : 'example.org',
  user            : 'bob',
  password        : 'secret'
});

pool.query('SELECT 1 + 1 AS solution', function(err, rows, fields) {
  if (err) throw err;

  console.log('The solution is: ', rows[0].solution);
});
```

Connections can be pooled to ease sharing a single connection, or managing
multiple connections.
连接池可以很容易的管理多条连接,并且共享单条连接.

```js
var mysql = require('mysql');
var pool  = mysql.createPool({
  host     : 'example.org',
  user     : 'bob',
  password : 'secret'
});

pool.getConnection(function(err, connection) {
  // connected! (unless `err` is set)
});
```

When you are done with a connection, just call `connection.release()` and the
connection will return to the pool, ready to be used again by someone else.

连接是使用完后可以调用`connection.release()`方法. 连接会返回连接池等待下次被调用.

```js
var mysql = require('mysql');
var pool  = mysql.createPool(...);

pool.getConnection(function(err, connection) {
  // Use the connection
  connection.query( 'SELECT something FROM sometable', function(err, rows) {
    // And done with the connection.
    connection.release();

    // Don't use the connection here, it has been returned to the pool.
  });
});
```

If you would like to close the connection and remove it from the pool, use
`connection.destroy()` instead. The pool will create a new connection the next
time one is needed.

如果你喜欢关闭连接后就从连接池删除掉它也可以, 下次调用时连接池会创建一个新的连接. 

Connections are lazily created by the pool. If you configure the pool to allow
up to 100 connections, but only ever use 5 simultaneously, only 5 connections
will be made. Connections are also cycled round-robin style, with connections
being taken from the top of the pool and returning to the bottom.

连接池里所有的连接时被动创建的. 比如你配置连接池最大连接数100. 但是最多只使用5个连接.
那么只有5个连接被被创建. 所有的池中连接被循环使用.连接池首先调用第一条连接,使用完后将它
排列到队列末尾

When a previous connection is retrieved from the pool, a ping packet is sent
to the server to check if the connection is still good.

当刚刚用完的连接返回连接池时, 会发送一个ping包给服务器检查连接是否正常.

## Pool options
## 连接池选项

Pools accept all the same options as a connection. When creating a new
connection, the options are simply passed to the connection constructor. In
addition to those options pools accept a few extras:

连接池接受创建连接时一样的选项. 当连接池创建连接时会直接把这部分选项直接传递给连接.
当然还有另外一部分的选项是配置连接池的:

* `acquireTimeout`: The milliseconds before a timeout occurs during the connection
  acquisition. This is slightly different from `connectTimeout`, because acquiring
  a pool connection does not always involve making a connection. (Default: `10000`)
* 请求超时时间,缺省10秒
* `waitForConnections`: Determines the pool's action when no connections are
  available and the limit has been reached. If `true`, the pool will queue the
  connection request and call it when one becomes available. If `false`, the
  pool will immediately call back with an error. (Default: `true`)
* 连接请求等待: 当连接池中连接数达到上限,并且没有空闲可用的连接时,如果该参数为`true`
  请求将会等待直到有一条连接可以使用. 如果为`false`则立刻返回一个错误.
* `connectionLimit`: The maximum number of connections to create at once.
  (Default: `10`)
  同一时刻可以使用的最大连接数,缺省为10.
* `queueLimit`: The maximum number of connection requests the pool will queue
  before returning an error from `getConnection`. If set to `0`, there is no
  limit to the number of queued connection requests. (Default: `0`)
  在`getConnection`返回错误之前,等待请求队列中可以排列的最大请求数,如果设为`0`则没有限制.
  缺省为`0`.

## Pool events
## 连接池事件

### connection

The pool will emit a `connection` event when a new connection is made within the pool. 
If you need to set session variables on the connection before it gets used, you can
listen to the `connection` event.

当一个新的连接在连接池中创建会触发一个`connection`事件.如果你在这个连接使用前为它设置一个
session属性. 你就可以监听这个`connection`事件了.

```js
pool.on('connection', function (connection) {
  connection.query('SET SESSION auto_increment_increment=1')
});
```

### enqueue

The pool will emit an `enqueue` event when a callback has been queued to wait for
an available connection.

当一个回调函数排队等待一个可用的连接时, 连接池将触发`enqueue`事件.

```js
pool.on('enqueue', function () {
  console.log('Waiting for available connection slot');
});
```

## Closing all the connections in a pool

When you are done using the pool, you have to end all the connections or the
Node.js event loop will stay active until the connections are closed by the
MySQL server. This is typically done if the pool is used in a script or when
trying to gracefully shutdown a server. To end all the connections in the
pool, use the `end` method on the pool:

当使用完一个连接池后,你需要关闭所有连接. 否则Node.js事件将保持在活动状态直到所有连接
被MySQL服务器关闭. 如果连接池中使用的脚本要安全的关闭服务器通常会这样做. 使用`end`方法
结束所有连接池中的连接.

```js
pool.end(function (err) {
  // all connections in the pool have ended
});
```

The `end` method takes an _optional_ callback that you can use to know once
all the connections have ended. The connections end _gracefully_, so all
pending queries will still complete and the time to end the pool will vary.

`end`方法有一个可选的回调函数.  让你可以一次知道是否所有的连接都正常关闭了.
所以所有的等待查询将在连接池关闭前陆续结束.

**Once `pool.end()` has been called, `pool.getConnection` and other operations
can no longer be performed**

一旦`pool.end()` 被调用了. `pool.getConnection`和其它操作将没有响应.


## PoolCluster

### 连接集群

PoolCluster provides multiple hosts connection. (group & retry & selector)

连接集群提供多个主机或者ip的连接.(群组,重连,选择器)

```js
// create
var poolCluster = mysql.createPoolCluster();

// add configurations
poolCluster.add(config); // anonymous group
poolCluster.add('MASTER', masterConfig);
poolCluster.add('SLAVE1', slave1Config);
poolCluster.add('SLAVE2', slave2Config);

// remove configurations
poolCluster.remove('SLAVE2'); // By nodeId
poolCluster.remove('SLAVE*'); // By target group : SLAVE1-2

// Target Group : ALL(anonymous, MASTER, SLAVE1-2), Selector : round-robin(default)
poolCluster.getConnection(function (err, connection) {});

// Target Group : MASTER, Selector : round-robin
poolCluster.getConnection('MASTER', function (err, connection) {});

// Target Group : SLAVE1-2, Selector : order
// If can't connect to SLAVE1, return SLAVE2. (remove SLAVE1 in the cluster)
poolCluster.on('remove', function (nodeId) {
  console.log('REMOVED NODE : ' + nodeId); // nodeId = SLAVE1 
});

poolCluster.getConnection('SLAVE*', 'ORDER', function (err, connection) {});

// of namespace : of(pattern, selector)
poolCluster.of('*').getConnection(function (err, connection) {});

var pool = poolCluster.of('SLAVE*', 'RANDOM');
pool.getConnection(function (err, connection) {});
pool.getConnection(function (err, connection) {});

// close all connections
poolCluster.end(function (err) {
  // all connections in the pool cluster have ended
});
```

## PoolCluster Option
## 连接集群选项

* `canRetry`: If `true`, `PoolCluster` will attempt to reconnect when connection fails. (Default: `true`)
* 如果为true,当连接失败后集群会尝试重新连接. 默认true
* `removeNodeErrorCount`: If connection fails, node's `errorCount` increases. 
  When `errorCount` is greater than `removeNodeErrorCount`, remove a node in the `PoolCluster`. (Default: `5`)
* 删除节的最大连点数, 如果连接失败,节点`errorCount`会增加. 当`errorCount`大于设置值时. 就从集群中删除这个节点.
* `restoreNodeTimeout`: If connection fails, specifies the number of milliseconds
  before another connection attempt will be made. If set to `0`, then node will be
  removed instead and never re-used. (Default: `0`)
* 如果一个连接失败, 在参数设置的时间内会试图创建新的连接. 如果设置为0则节点会被删除不再使用. 缺省0
* `defaultSelector`: The default selector. (Default: `RR`)
  * `RR`: Select one alternately. (Round-Robin)
  * `RANDOM`: Select the node by random function.
  * `ORDER`: Select the first node available unconditionally.
* 缺省选择器(默认`RR`)
  * `RR`: 循环选择一个
  * `RANDOM`: 通过随机方法选择节点
  * `ORDER`: 无条件选择第一个可用的节点

```js
var clusterConfig = {
  removeNodeErrorCount: 1, // Remove the node immediately when connection fails.
  defaultSelector: 'ORDER'
};

var poolCluster = mysql.createPoolCluster(clusterConfig);
```

## Switching users and altering connection state
## 改变连接用户和状态 `changeUser`命令

MySQL offers a changeUser command that allows you to alter the current user and
other aspects of the connection without shutting down the underlying socket:

MySQL支持`changeUser`命令,允许你在不关闭底层连接的情况下更改当前连接的用户或者其它选项.

```js
connection.changeUser({user : 'john'}, function(err) {
  if (err) throw err;
});
```

The available options for this feature are:

可以改变的参数包括

* `user`: The name of the new user (defaults to the previous one).
* `password`: The password of the new user (defaults to the previous one).
* `charset`: The new charset (defaults to the previous one).
* `database`: The new database (defaults to the previous one).

A sometimes useful side effect of this functionality is that this function also
resets any connection state (variables, transactions, etc.).

这个功能在需要重置连接状态的时候也非常有用(变量,事务,等).

Errors encountered during this operation are treated as fatal connection errors
by this module.

使用这个功能可以应用在重要的连接发生错误时的错误处理.

## Server disconnects
## 服务器断开
You may lose the connection to a MySQL server due to network problems, the
server timing you out, the server being restarted, or crashing. All of these
events are considered fatal errors, and will have the `err.code =
'PROTOCOL_CONNECTION_LOST'`.  See the [Error Handling](#error-handling) section
for more information. 

你许会因为网络问题失去MySQL服务器的连接,服务器超时,服务器重启,服务器崩溃,所有这些
事情都会导致严重的错误.并且会产生`err.code ='PROTOCOL_CONNECTION_LOST'`. 参见
[Error Handling] 部分.

Re-connecting a connection is done by establishing a new connection. Once
terminated, an existing connection object cannot be re-connected by design.

重新连接时重新建立一条新的连接. 如果连接中断以后原来的连接是没办法重连的.

With Pool, disconnected connections will be removed from the pool freeing up
space for a new connection to be created on the next getConnection call.

在连接池中. 所有断开的连接会从连接池空释放空间给下一个getConnection方法调用时创建的新连接.

## Performing queries
## 查询执行
The most basic way to perform a query is to call the `.query()` method on an object
(like a `Connection` or `Pool` instance).

一般要执行查询是通过`Connection`或者`Pool`的`.query()`方法.

The simplest form of .`query()` is `.query(sqlString, callback)`, where a SQL string
is the first argument and the second is a callback:

最简单的是使用`.query(sqlString,callback)`,第一个参数是sql语句字符串. 第二个回调函数.

```js
connection.query('SELECT * FROM `books` WHERE `author` = "David"', function (error, results, fields) {
  // error will be an Error if one occurred during the query
  // results will contain the results of the query
  // fields will contain information about the returned results fields (if any)
});
```

The second form `.query(sqlString, values, callback)` comes when using
placeholder values (see [escaping query values](#escaping-query-values)):

当使用占位符来查询时使用第二种方法`.query(sqlString,values,callback)` 参见[escaping query values](#escaping-query-values)

```js
connection.query('SELECT * FROM `books` WHERE `author` = ?', ['David'], function (error, results, fields) {
  // error will be an Error if one occurred during the query
  // results will contain the results of the query
  // fields will contain information about the returned results fields (if any)
});
```

The third form `.query(options, callback)` comes when using various advanced
options on the query, like [escaping query values](#escaping-query-values),
[joins with overlapping column names](#joins-with-overlapping-column-names),
[timeouts](#timeout), and [type casting](#type-casting).

第三种方法`.query(options,callback)` 查询是使用高级选项 option 像[escaping query values](#escaping-query-values),[joins with overlapping column names](#joins-with-overlapping-column-names),
[timeouts](#timeout), 和 [type casting](#type-casting).

```js
connection.query({
  sql: 'SELECT * FROM `books` WHERE `author` = ?',
  timeout: 40000, // 40s
  values: ['David']
}, function (error, results, fields) {
  // error will be an Error if one occurred during the query
  // results will contain the results of the query
  // fields will contain information about the returned results fields (if any)
});
```

Note that a combination of the second and third forms can be used where the
placeholder values are passes as an argument and not in the options object.
The `values` argument will override the `values` in the option object.

注意当第二和第三种方法组合位符可以作为查询参数. 但是不能在查询选项options对象中出现.

```js
connection.query({
    sql: 'SELECT * FROM `books` WHERE `author` = ?',
    timeout: 40000, // 40s
  },
  ['David'],
  function (error, results, fields) {
    // error will be an Error if one occurred during the query
    // results will contain the results of the query
    // fields will contain information about the returned results fields (if any)
  }
);
```

## Escaping query values
## 加密查询数据(Escaping 为隐藏逃逸的意思,也有回避问题的意思.暂时找不到合适的翻译 在这一节我理解应该译为加密,下一节应该是验证代码合法性的意思.暂时如此)

In order to avoid SQL Injection attacks, you should always escape any user
provided data before using it inside a SQL query. You can do so using the
`mysql.escape()`, `connection.escape()` or `pool.escape()` methods:

为了避免SQL注入攻击,你应该把用户提供的数据始终在查询语句中加密. 你可以使用
`mysql.escape()`,`connection.escape()`和`escape()`方法如下:

```js
var userId = 'some user provided value';
var sql    = 'SELECT * FROM users WHERE id = ' + connection.escape(userId);
connection.query(sql, function(err, results) {
  // ...
});
```

Alternatively, you can use `?` characters as placeholders for values you would
like to have escaped like this:

或者你可以使用`?`字符作为占位符来加密敏感数据就像这样:

```js
connection.query('SELECT * FROM users WHERE id = ?', [userId], function(err, results) {
  // ...
});
```

This looks similar to prepared statements in MySQL, however it really just uses
the same `connection.escape()` method internally.

看起来查询语句简单多了. 实际是和`connection.escape()`内部机制是一样的.

**Caution** This also differs from prepared statements in that all `?` are
replaced, even those contained in comments and strings.

**警告**: 所有的`?`都会被替换.包括注释和字符串中的.

Different value types are escaped differently, here is how:

不同类型的数据加密后结果不通.

* Numbers are left untouched  //数字不会有变化
* Booleans are converted to `true` / `false`
* Date objects are converted to `'YYYY-mm-dd HH:ii:ss'` strings
* Buffers are converted to hex strings, e.g. `X'0fa5'`
* Strings are safely escaped
* Arrays are turned into list, e.g. `['a', 'b']` turns into `'a', 'b'`
* Nested arrays are turned into grouped lists (for bulk inserts), e.g. `[['a',
  'b'], ['c', 'd']]` turns into `('a', 'b'), ('c', 'd')`
* Objects are turned into `key = 'val'` pairs for each enumerable property on
  the object. If the property's value is a function, it is skipped; if the
  property's value is an object, toString() is called on it and the returned
  value is used.
* `undefined` / `null` are converted to `NULL`
* `NaN` / `Infinity` are left as-is. MySQL does not support these, and trying
  to insert them as values will trigger MySQL errors until they implement
  support.
* 不支持`NaN` 或者 `Infinity`类型. 

If you paid attention, you may have noticed that this escaping allows you
to do neat things like this:

如果你细心你会发现,这种方法能让你的代码更整洁清爽. 例如:

```js
var post  = {id: 1, title: 'Hello MySQL'};
var query = connection.query('INSERT INTO posts SET ?', post, function(err, result) {
  // Neat!
});
console.log(query.sql); // INSERT INTO posts SET `id` = 1, `title` = 'Hello MySQL'

```

If you feel the need to escape queries by yourself, you can also use the escaping
function directly:

如果你觉得喜欢自己直接写加密方法那也没问题:

```js
var query = "SELECT * FROM posts WHERE title=" + mysql.escape("Hello MySQL");

console.log(query); // SELECT * FROM posts WHERE title='Hello MySQL'
```

## Escaping query identifiers
## 验证查询标识符 (这一节Escaping这个词应该取规避错误,我理解是验证的意思.)

If you can't trust an SQL identifier (database / table / column name) because it is
provided by a user, you should escape it with `mysql.escapeId(identifier)`,
`connection.escapeId(identifier)` or `pool.escapeId(identifier)` like this:

如果你不能信任一个SQL标识符,因为它有可能是来自用户提供的数据. 那么你可以使用
`mysql.escapeId(identifier)`,`connection.escapeId(identifier)` or `pool.escapeId(identifier)` 方法像这样:

```js
var sorter = 'date';
var sql    = 'SELECT * FROM posts ORDER BY ' + connection.escapeId(sorter);
connection.query(sql, function(err, results) {
  // ...
});
```

It also supports adding qualified identifiers. It will escape both parts.

它同时支持常量和变量一起. 同时对两部分起作用.

```js
var sorter = 'date';
var sql    = 'SELECT * FROM posts ORDER BY ' + connection.escapeId('posts.' + sorter);
connection.query(sql, function(err, results) {
  // ...
});
```

Alternatively, you can use `??` characters as placeholders for identifiers you would
like to have escaped like this:

另外,你可以使用`??` 字符作为占位符的标示.

```js
var userId = 1;
var columns = ['username', 'email'];
var query = connection.query('SELECT ?? FROM ?? WHERE id = ?', [columns, 'users', userId], function(err, results) {
  // ...
});

console.log(query.sql); // SELECT `username`, `email` FROM `users` WHERE id = 1
```
**Please note that this last character sequence is experimental and syntax might change**

**最后的字符排列还在实验,语法将来可能会变化**

When you pass an Object to `.escape()` or `.query()`, `.escapeId()` is used to avoid SQL injection in object keys.

当你用对象作为查询参数,`.escapeId()`可以用来防止对象键值的SQL注入攻击.
(JSON对象键值是可以任意原型扩展.攻击者可以随意在JSON对象中加入`.method`这样的方法.)

### Preparing Queries
### 查询准备

You can use mysql.format to prepare a query with multiple insertion points, utilizing the proper escaping for ids and values. A simple example of this follows:

你可以使用mysql.format方法来使用多点查询,利用正确的方法防止入侵检测保护数据.例如:

```js
var sql = "SELECT * FROM ?? WHERE ?? = ?";
var inserts = ['users', 'id', userId];
sql = mysql.format(sql, inserts);
```

Following this you then have a valid, escaped query that you can then send to the database safely. This is useful if you are looking to prepare the query before actually sending it to the database. As mysql.format is exposed from SqlString.format you also have the option (but are not required) to pass in stringifyObject and timezone, allowing you provide a custom means of turning objects into strings, as well as a location-specific/timezone-aware Date.

接下来你应该做一个验证方法,保护你要发送到数据库的查询数据安全. 在发送到数据库前准备查询语句是非常有用.
就想mysql.format是暴露的, SqlString.format也有可选项(但不是必选)来传递StringifyObject和时区,允许你提供一个自定义转换对象在字符串. 以及特定时间日期格式.

### Custom format
### 自定义格式

If you prefer to have another type of query escape format, there's a connection configuration option you can use to define a custom format function. You can access the connection object if you want to use the built-in `.escape()` or any other connection function.

如果你希望使用其它类型的查询格式, 你可以使用内置`.escape()` 或者其它方法在连接字符串的配置里自定义格式.

Here's an example of how to implement another format:

这是一个实现了自定义格式的例子:

```js
connection.config.queryFormat = function (query, values) {
  if (!values) return query;
  return query.replace(/\:(\w+)/g, function (txt, key) {
    if (values.hasOwnProperty(key)) {
      return this.escape(values[key]);
    }
    return txt;
  }.bind(this));
};

connection.query("UPDATE posts SET title = :title", { title: "Hello MySQL" });
```

## Getting the id of an inserted row
## 如何得到已经插入数据的id

If you are inserting a row into a table with an auto increment primary key, you
can retrieve the insert id like this:

如果你插入的一行数据是一个主键自增长的表. 你可以像这样返回插入的主键.

```js
connection.query('INSERT INTO posts SET ?', {title: 'test'}, function(err, result) {
  if (err) throw err;

  console.log(result.insertId);
});
```

When dealing with big numbers (above JavaScript Number precision limit), you should
consider enabling `supportBigNumbers` option to be able to read the insert id as a
string, otherwise it will throw.

如果你正字处理长数字类型(超过了Javascript允许范围的), 你应该考虑启用`supportBigNumbers`这个选项
来以字符串的方式来读取插入的id值. 否则将会报错.

This option is also required when fetching big numbers from the database, otherwise
you will get values rounded to hundreds or thousands due to the precision limit.

这个选项在从数据库中读取长数字类型的的时候同样需要, 否则由于精度你得到的数据将会有随机几百到
几千的误差.

## Getting the number of affected rows
## 如何得到执行记录数

You can get the number of affected rows from an insert, update or delete statement.

你可以通过下面方法在insert update和delete语句中得到执行行数.

```js
connection.query('DELETE FROM posts WHERE title = "wrong"', function (err, result) {
  if (err) throw err;

  console.log('deleted ' + result.affectedRows + ' rows');
})
```

## Getting the number of changed rows
## 如何得到修改记录数

You can get the number of changed rows from an update statement.

"changedRows" differs from "affectedRows" in that it does not count updated rows
whose values were not changed.

你可以通过下面方法得到修改记录的数量. 修改记录数和执行记录数不同碍于不统计那些执行后值没有变化的记录.

```js
connection.query('UPDATE posts SET ...', function (err, result) {
  if (err) throw err;

  console.log('changed ' + result.changedRows + ' rows');
})
```

## Getting the connection ID
## 如何得到connection ID和thread ID

You can get the MySQL connection ID ("thread ID") of a given connection using the `threadId`
property.

你可以通过一个连接的`threadId`属性来得到连接MySQL的connection ID.也就是"thread ID".

```js
connection.connect(function(err) {
  if (err) throw err;
  console.log('connected as id ' + connection.threadId);
});
```

## Executing queries in parallel
## 如何操作并行查询

The MySQL protocol is sequential, this means that you need multiple connections
to execute queries in parallel. You can use a Pool to manage connections, one
simple approach is to create one connection per incoming http request.

MySQL执行的通信协议是同步的,这意味你需要多个连接来执行并行查询. 你可以使用连接池来管理连接.
简单的方法就是为每个http请求创建一个连接.

## Streaming query rows
## 如何操作数据流.

Sometimes you may want to select large quantities of rows and process each of
them as they are received. This can be done like this:

有时候你要以流的方式从每条记录中取出一个比较大的数据. 你可以像下面这样做

```js
var query = connection.query('SELECT * FROM posts');
query
  .on('error', function(err) {
    // Handle error, an 'end' event will be emitted after this as well
  })
  .on('fields', function(fields) {
    // the field packets for the rows to follow
  })
  .on('result', function(row) {
    // Pausing the connnection is useful if your processing involves I/O
    // 当你在执行I/O操作前在这里暂停connection是非常必要的.
    connection.pause();

    processRow(row, function() {
      connection.resume();
    });
  })
  .on('end', function() {
    // all rows have been received
  });
```

Please note a few things about the example above:

你能发现上面例子的一些优点.

* Usually you will want to receive a certain amount of rows before starting to
  throttle the connection using `pause()`. This number will depend on the
  amount and size of your rows.
 
  通常你可以在`pause()`操作前收到总共的记录条数. 这个数字取决你所有记录的大小.

* `pause()` / `resume()` operate on the underlying socket and parser. You are
  guaranteed that no more `'result'` events will fire after calling `pause()`.

  `pause()`/`resume()`处理底层的数据的时候保证不会有`result`事件触发.

* You MUST NOT provide a callback to the `query()` method when streaming rows.

  你不用为流数据提供回调函数.

* The `'result'` event will fire for both rows as well as OK packets
  confirming the success of a INSERT/UPDATE query.

  当所有的记录确认成功操作后`result`事件被发送.

* It is very important not to leave the result paused too long, or you may
  encounter `Error: Connection lost: The server closed the connection.`
  The time limit for this is determined by the
  [net_write_timeout setting](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_net_write_timeout)
  on your MySQL server.

  最重要的是~ 不要让result事件暂停的太久,否则可能会发生 `Error: Connection lost: The server closed the connection.` 与服务器失去联系, 服务器关闭了了连接. 时间限制决定于你的MySQL服务器设置.

Additionally you may be interested to know that it is currently not possible to
stream individual row columns, they will always be buffered up entirely. If you
have a good use case for streaming large fields to and from MySQL, I'd love to
get your thoughts and contributions on this.

另外现在对单独的列的流操作还没有实现.现在只能全部靠缓存处理,如果你有好的处理MySQL大字段的经验和例子,我非常希望能知道你方案能够用在这里.

### Piping results with Streams2
### 
The query object provides a convenience method `.stream([options])` that wraps
query events into a [Readable](http://nodejs.org/api/stream.html#stream_class_stream_readable)
[Streams2](http://blog.nodejs.org/2012/12/20/streams2/) object. This
stream can easily be piped downstream and provides automatic pause/resume,
based on downstream congestion and the optional `highWaterMark`. The
`objectMode` parameter of the stream is set to `true` and cannot be changed
(if you need a byte stream, you will need to use a transform stream, like
[objstream](https://www.npmjs.com/package/objstream) for example).

这段暂时不理解

For example, piping query results into another stream (with a max buffer of 5
objects) is simply:

样例, 管道查询结果到另外一个流中(最多5个缓冲对象).

```js
connection.query('SELECT * FROM posts')
  .stream({highWaterMark: 5})
  .pipe(...);
```

## Multiple statement queries

Support for multiple statements is disabled for security reasons (it allows for
SQL injection attacks if values are not properly escaped). To use this feature
you have to enable it for your connection:

支持多语句不考虑安全因素(这样允许SQL注入式攻击). 要使用这个功能你的连接要启用这个选项.

```js
var connection = mysql.createConnection({multipleStatements: true});
```

Once enabled, you can execute multiple statement queries like any other query:

一旦启用这个选项,你可以在一个查询中输入多条SQL语句.

```js
connection.query('SELECT 1; SELECT 2', function(err, results) {
  if (err) throw err;

  // `results` is an array with one element for every statement in the query:
  console.log(results[0]); // [{1: 1}]
  console.log(results[1]); // [{2: 2}]
});
```

Additionally you can also stream the results of multiple statement queries:

另外你可以用流的方式处理多语句查询的结果. 例如:

```js
var query = connection.query('SELECT 1; SELECT 2');

query
  .on('fields', function(fields, index) {
    // the fields for the result rows that follow
  })
  .on('result', function(row, index) {
    // index refers to the statement this result belongs to (starts at 0)
    // index是查询结果的序列.从0开始.
  });
```

If one of the statements in your query causes an error, the resulting Error
object contains a `err.index` property which tells you which statement caused
it. MySQL will also stop executing any remaining statements when an error
occurs.

如果其中一条查询导致了一个错误. 返回的错误对象包含`err.index`属性鞥告诉你是哪条语句导致的错误.
当一个错误发生后MySQL将会停止执行剩余的查询语句.

Please note that the interface for streaming multiple statement queries is
experimental and I am looking forward to feedback on it.

**注意, 多语句查询的流处理宫功能也是实验阶段我会继续关注研究者一部分**

## Stored procedures

You can call stored procedures from your queries as with any other mysql driver.
If the stored procedure produces several result sets, they are exposed to you
the same way as the results for multiple statement queries.

你可以使用其它mysql驱动在你的查询语句中调用存储过程. 如果存储过程处理的结果也多个结果集. 你可以使用多语句查询一样的方法来处理结果.

## Joins with overlapping column names
## 使用join方法时同名字段如何处理

When executing joins, you are likely to get result sets with overlapping column
names.

当执行joins的语句时,经常会得到查询结果集里有相同列名的情况.

By default, node-mysql will overwrite colliding column names in the
order the columns are received from MySQL, causing some of the received values
to be unavailable.

缺省node-mysql会安装从MySQL返回的顺序重新排列列名. 这样会导致收到数据不可用.

However, you can also specify that you want your columns to be nested below
the table name like this:

不过你也可以使用表名和列名嵌套的方法来指定列名.如下:

```js
var options = {sql: '...', nestTables: true};
connection.query(options, function(err, results) {
  /* results will be an array like this now:
  [{
    table1: {
      fieldA: '...',
      fieldB: '...',
    },
    table2: {
      fieldA: '...',
      fieldB: '...',
    },
  }, ...]
  */
});
```

Or use a string separator to have your results merged.

```js
var options = {sql: '...', nestTables: '_'};
connection.query(options, function(err, results) {
  /* results will be an array like this now:
  [{
    table1_fieldA: '...',
    table1_fieldB: '...',
    table2_fieldA: '...',
    table2_fieldB: '...',
  }, ...]
  */
});
```

## Transactions
## 事务处理

Simple transaction support is available at the connection level:

在连接的层次可以支持简单的失误处理操作.

```js
connection.beginTransaction(function(err) {
  if (err) { throw err; }
  connection.query('INSERT INTO posts SET title=?', title, function(err, result) {
    if (err) {
      return connection.rollback(function() {
        throw err;
      });
    }

    var log = 'Post ' + result.insertId + ' added';

    connection.query('INSERT INTO log SET data=?', log, function(err, result) {
      if (err) {
        return connection.rollback(function() {
          throw err;
        });
      }  
      connection.commit(function(err) {
        if (err) {
          return connection.rollback(function() {
            throw err;
          });
        }
        console.log('success!');
      });
    });
  });
});
```
Please note that beginTransaction(), commit() and rollback() are simply convenience
functions that execute the START TRANSACTION, COMMIT, and ROLLBACK commands respectively.
It is important to understand that many commands in MySQL can cause an implicit commit,
as described [in the MySQL documentation](http://dev.mysql.com/doc/refman/5.5/en/implicit-commit.html)

要注意使用beginTransaction(), commit() and rollback()这些功能来执行对应的命令是很方便. 但是重要的是要知道太多的命令在MySQL里会导致隐式的commit. 详见:[in the MySQL documentation](http://dev.mysql.com/doc/refman/5.5/en/implicit-commit.html)

## Ping
## Ping

A ping packet can be sent over a connection using the `connection.ping` method. This
method will send a ping packet to the server and when the server responds, the callback
will fire. If an error occurred, the callback will fire with an error argument.

通过connection.ping方法可发送一个ping包给服务器.当服务器响应以后会触发callback函数.如何出现错误
callback函数会把错误发送给error参数. 

```js
connection.ping(function (err) {
  if (err) throw err;
  console.log('Server responded to ping');
})
```

## Timeouts
## 超时

Every operation takes an optional inactivity timeout option. This allows you to
specify appropriate timeouts for operations. It is important to note that these
timeouts are not part of the MySQL protocol, and rather timeout operations through
the client. This means that when a timeout is reached, the connection it occurred
on will be destroyed and no further operations can be performed.

每个操作都有一个闲置超时选项,它能让你指定一个合适的操作超时时间. 重点是这个超时时间不是MySQL协议的部分.
而是客户端设置的. 如何连接闲置时间到达设置的限制,连接将会关闭,再也不能操作这个安静.

```js
// Kill query after 60s
connection.query({sql: 'SELECT COUNT(*) AS count FROM big_table', timeout: 60000}, function (err, rows) {
  if (err && err.code === 'PROTOCOL_SEQUENCE_TIMEOUT') {
    throw new Error('too long to count table rows!');
  }

  if (err) {
    throw err;
  }

  console.log(rows[0].count + ' rows');
});
```

## Error handling
## 错误处理

This module comes with a consistent approach to error handling that you should
review carefully in order to write solid applications.

这个模块是为了方便统一进行错误处理,让你检查方便,以致可以写出更强壮的应用程序.

All errors created by this module are instances of the JavaScript [Error][]
object. Additionally they come with two properties:

所有错误都是Javascriptp[Error][]对象创建的实例. 另外还为他们还添加了两个属性.

* `err.code`: Either a [MySQL server error][] (e.g.
  `'ER_ACCESS_DENIED_ERROR'`), a node.js error (e.g. `'ECONNREFUSED'`) or an
  internal error (e.g.  `'PROTOCOL_CONNECTION_LOST'`).
* `err.fatal`: Boolean, indicating if this error is terminal to the connection
  object.

[Error]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Error
[MySQL server error]: http://dev.mysql.com/doc/refman/5.5/en/error-messages-server.html

Fatal errors are propagated to *all* pending callbacks. In the example below, a
fatal error is triggered by trying to connect to an invalid port. Therefore the
error object is propagated to both pending callbacks:

致命错误会传播给所有正在挂起的回调函数. 例如:一个尝试连接错误端口的致命错误. 错误对象传播给了每个正在挂起的回调函数.

```js
var connection = require('mysql').createConnection({
  port: 84943, // WRONG PORT
});

connection.connect(function(err) {
  console.log(err.code); // 'ECONNREFUSED'
  console.log(err.fatal); // true
});

connection.query('SELECT 1', function(err) {
  console.log(err.code); // 'ECONNREFUSED'
  console.log(err.fatal); // true
});
```

Normal errors however are only delegated to the callback they belong to.  So in
the example below, only the first callback receives an error, the second query
works as expected:

普通错误,仅仅返回给他们所属的回调函数. 所以下面例子只有第一个回调函数收到错误. 第二个工作正常.

```js
connection.query('USE name_of_db_that_does_not_exist', function(err, rows) {
  console.log(err.code); // 'ER_BAD_DB_ERROR'
});

connection.query('SELECT 1', function(err, rows) {
  console.log(err); // null
  console.log(rows.length); // 1
});
```

Last but not least: If a fatal errors occurs and there are no pending
callbacks, or a normal error occurs which has no callback belonging to it, the
error is emitted as an `'error'` event on the connection object. This is
demonstrated in the example below:

最后很重要的是, 如何致命错误发生却没有挂起的回调函数,普通错误发生也没有他们所属的回调函数.这个错误会触发connection的`'error'`事件. 例如:
  
```js
connection.on('error', function(err) {
  console.log(err.code); // 'ER_BAD_DB_ERROR'
});

connection.query('USE name_of_db_that_does_not_exist');
```

Note: `'error'` events are special in node. If they occur without an attached
listener, a stack trace is printed and your process is killed.

**tl;dr:** This module does not want you to deal with silent failures. You
should always provide callbacks to your method calls. If you want to ignore
this advice and suppress unhandled errors, you can do this:

```js
// I am Chuck Norris:
connection.on('error', function() {});
```

## Exception Safety

This module is exception safe. That means you can continue to use it, even if
one of your callback functions throws an error which you're catching using
'uncaughtException' or a domain.

## Type casting

For your convenience, this driver will cast mysql types into native JavaScript
types by default. The following mappings exist:

### Number

* TINYINT
* SMALLINT
* INT
* MEDIUMINT
* YEAR
* FLOAT
* DOUBLE

### Date

* TIMESTAMP
* DATE
* DATETIME

### Buffer

* TINYBLOB
* MEDIUMBLOB
* LONGBLOB
* BLOB
* BINARY
* VARBINARY
* BIT (last byte will be filled with 0 bits as necessary)

### String

**Note** text in the binary character set is returned as `Buffer`, rather
than a string.

* CHAR
* VARCHAR
* TINYTEXT
* MEDIUMTEXT
* LONGTEXT
* TEXT
* ENUM
* SET
* DECIMAL (may exceed float precision)
* BIGINT (may exceed float precision)
* TIME (could be mapped to Date, but what date would be set?)
* GEOMETRY (never used those, get in touch if you do)

It is not recommended (and may go away / change in the future) to disable type
casting, but you can currently do so on either the connection:

```js
var connection = require('mysql').createConnection({typeCast: false});
```

Or on the query level:

```js
var options = {sql: '...', typeCast: false};
var query = connection.query(options, function(err, results) {

});
```

You can also pass a function and handle type casting yourself. You're given some
column information like database, table and name and also type and length. If you
just want to apply a custom type casting to a specific type you can do it and then
fallback to the default. Here's an example of converting `TINYINT(1)` to boolean:

```js
connection.query({
  sql: '...',
  typeCast: function (field, next) {
    if (field.type == 'TINY' && field.length == 1) {
      return (field.string() == '1'); // 1 = true, 0 = false
    }
    return next();
  }
});
```
__WARNING: YOU MUST INVOKE the parser using one of these three field functions in your custom typeCast callback. They can only be called once.( see #539 for discussion)__

```
field.string()
field.buffer()
field.geometry()
```
are aliases for
```
parser.parseLengthCodedString()
parser.parseLengthCodedBuffer()
parser.parseGeometryValue()
```
__You can find which field function you need to use by looking at: [RowDataPacket.prototype._typeCast](https://github.com/felixge/node-mysql/blob/master/lib/protocol/packets/RowDataPacket.js#L41)__


## Connection Flags

If, for any reason, you would like to change the default connection flags, you
can use the connection option `flags`. Pass a string with a comma separated list
of items to add to the default flags. If you don't want a default flag to be used
prepend the flag with a minus sign. To add a flag that is not in the default list,
just write the flag name, or prefix it with a plus (case insensitive).

**Please note that some available flags that are not supported (e.g.: Compression),
are still not allowed to be specified.**

### Example

The next example blacklists FOUND_ROWS flag from default connection flags.

```js
var connection = mysql.createConnection("mysql://localhost/test?flags=-FOUND_ROWS");
```

### Default Flags

The following flags are sent by default on a new connection:

- `CONNECT_WITH_DB` - Ability to specify the database on connection.
- `FOUND_ROWS` - Send the found rows instead of the affected rows as `affectedRows`.
- `IGNORE_SIGPIPE` - Old; no effect.
- `IGNORE_SPACE` - Let the parser ignore spaces before the `(` in queries.
- `LOCAL_FILES` - Can use `LOAD DATA LOCAL`.
- `LONG_FLAG`
- `LONG_PASSWORD` - Use the improved version of Old Password Authentication.
- `MULTI_RESULTS` - Can handle multiple resultsets for COM_QUERY.
- `ODBC` Old; no effect.
- `PROTOCOL_41` - Uses the 4.1 protocol.
- `PS_MULTI_RESULTS` - Can handle multiple resultsets for COM_STMT_EXECUTE.
- `RESERVED` - Old flag for the 4.1 protocol.
- `SECURE_CONNECTION` - Support native 4.1 authentication.
- `TRANSACTIONS` - Asks for the transaction status flags.

In addition, the following flag will be sent if the option `multipleStatements`
is set to `true`:

- `MULTI_STATEMENTS` - The client may send multiple statement per query or
  statement prepare.

### Other Available Flags

There are other flags available. They may or may not function, but are still
available to specify.

- COMPRESS
- INTERACTIVE
- NO_SCHEMA
- PLUGIN_AUTH
- REMEMBER_OPTIONS
- SSL
- SSL_VERIFY_SERVER_CERT

## Debugging and reporting problems

If you are running into problems, one thing that may help is enabling the
`debug` mode for the connection:

```js
var connection = mysql.createConnection({debug: true});
```

This will print all incoming and outgoing packets on stdout. You can also restrict debugging to
packet types by passing an array of types to debug:

```js
var connection = mysql.createConnection({debug: ['ComQueryPacket', 'RowDataPacket']});
```

to restrict debugging to the query and data packets.

If that does not help, feel free to open a GitHub issue. A good GitHub issue
will have:

* The minimal amount of code required to reproduce the problem (if possible)
* As much debugging output and information about your environment (mysql
  version, node version, os, etc.) as you can gather.

## Running tests

The test suite is split into two parts: unit tests and integration tests.
The unit tests run on any machine while the integration tests require a
MySQL server instance to be setup.

### Running unit tests

```sh
$ FILTER=unit npm test
```

### Running integration tests

Set the environment variables `MYSQL_DATABASE`, `MYSQL_HOST`, `MYSQL_PORT`,
`MYSQL_USER` and `MYSQL_PASSWORD`. Then run `npm test`.

For example, if you have an installation of mysql running on localhost:3306
and no password set for the `root` user, run:

```sh
$ mysql -u root -e "CREATE DATABASE IF NOT EXISTS node_mysql_test"
$ MYSQL_HOST=localhost MYSQL_PORT=3306 MYSQL_DATABASE=node_mysql_test MYSQL_USER=root MYSQL_PASSWORD= FILTER=integration npm test
```

## Todo

* Prepared statements
* Support for encodings other than UTF-8 / ASCII

[npm-image]: https://img.shields.io/npm/v/mysql.svg
[npm-url]: https://npmjs.org/package/mysql
[node-version-image]: http://img.shields.io/node/v/mysql.svg
[node-version-url]: http://nodejs.org/download/
[travis-image]: https://img.shields.io/travis/felixge/node-mysql/master.svg?label=linux
[travis-url]: https://travis-ci.org/felixge/node-mysql
[appveyor-image]: https://img.shields.io/appveyor/ci/dougwilson/node-mysql/master.svg?label=windows
[appveyor-url]: https://ci.appveyor.com/project/dougwilson/node-mysql
[coveralls-image]: https://img.shields.io/coveralls/felixge/node-mysql/master.svg
[coveralls-url]: https://coveralls.io/r/felixge/node-mysql?branch=master
[downloads-image]: https://img.shields.io/npm/dm/mysql.svg
[downloads-url]: https://npmjs.org/package/mysql
