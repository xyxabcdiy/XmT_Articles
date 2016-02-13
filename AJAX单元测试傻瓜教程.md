Ajax 请求经常容易发生错误，客户端发送的数据出问题，服务器端返回的数据有误都会导致 Ajax 请求错误。你不能保证与服务器的连接总是工作正常。Ajax请求需要将用户的输入发送给服务器并返回服务器响应，因此，对于数据的正确处理至关重要。
但是由于Ajax是异步的，测试它的同时必须保证独立性，我们如何才能在Ajax与服务器进行通信的时候对其进行单元测试呢？
不要怕，让我们看一看接下来的例子，学习一下如何对Ajax请求进行单元测试。
##设置
在我们开始单元测试之前，我们需要安装几个必须的工具。
- 新建一个目录，用来存放必要的文件
- 使用 `npm install mocha chai sinon` 安装 `Mocha`, `Chai`, `Sinon`

###测试运行器
为了使事情简单，我们将直接在浏览器中运行测试。如果你更喜欢基于命令行的测试的话，测试运行的结果也将会和浏览器中的结果完全一致。
我们将会使用以下的文件作为测试运行器。我将其命名为`test.html`。

```
<!DOCTYPE html>
<html>
    <head>
        <title>Mocha Tests</title>
        <link rel="stylesheet" href="node_modules/mocha/mocha.css">
    </head>
    <body>
        <div id="mocha"></div>
        <script src="node_modules/mocha/mocha.js"></script>
        <script src="node_modules/sinon/pkg/sinon-1.12.2.js"></script>
        <script src="node_modules/chai/chai.js"></script>
        <script>mocha.setup('bdd')</script>
        <script src="testApi.js"></script>
        <script src="test.js"></script>
        <script>
            mocha.run();
        </script>
    </body>
</html>
```

注意`mocha.js`,`mocha.css`,`sinon-1.17.1.js`,`chai.js`的文件路径。因为我是用`npm`安装它们的，所以它们都在`node_modules`目录下。对于 `Sinon`，你可能需要更改文件名来匹配安装的版本，我在这里使用的是版本是1.17.1。同时也注意`testApi.js`,`test.js`这两个文件，这两个文件作为我的实例模块和测试用例，我将会在接下来逐一介绍它们。

###实例模块
接下来，我们将创建一个基础的模块，该模块将会发送一些Ajax请求。我们将用它来向你们展示如何对Ajax进行单元测试。
我们将该文件命名为`testApi.js`。

```
var testApi = {
    get: function(callback) {
        var xhr = new XMLHttpRequest();
        xhr.open('GET', 'http://jsonplaceholder.typicode.com/posts/1', true);

        xhr.onreadystatechange = function() {
            if(xhr.readyState == 4) {
                if(xhr.status == 200) {
                    callback(null, JSON.parse(xhr.responseText));
                }
                else {
                    callback(xhr.status);
                }
            }
        };

        xhr.send();
    },

    post: function(data, callback) {
        var xhr = new XMLHttpRequest();
        xhr.open('POST', 'http://jsonplaceholder.typicode.com/posts', true);

        xhr.onreadystatechange = function() {
            if(xhr.readyState == 4) {
                callback();
            }
        };

        xhr.send(JSON.stringify(data));
    }
};
```

这段代码开起来应该很熟悉，我们写了两个函数，一个函数中使用`GET`方法获取数据，另一个函数中使用`POST`方法向服务器发送数据，这些都是最普通的 Ajax请求。我们在这里使用了[JSONPlaceholder API](http://http://jsonplaceholder.typicode.com/ "JSONPlaceholder API")，它是一个免费的在线REST服务，你可以使用它提供的模拟数据，非常适合快速测试。

###测试用例框架
之后我们需要创建一个框架，在里面添加每个情景的测试集。该文件被命名为`test.js`。

```
chai.should();

describe('TestAPI', function() {
  //Tests etc. go here
});
```

`Mocha`中使用`describe`来创建一个测试用例，该测试用例便是我们添加测试代码的地方。
`chai.should()`将允许我们使用 "should style" 断言。这意味着我们可以轻松的验证我们的测试结果，如：`someValue.should.equal(12345)`。

##测试GET请求
我们的示例模块中有一个`get`函数，该函数可以被用来加载服务器传送过来的数据。我们将会创建一个测试函数来证明由服务器取到数据是一个JSON数据。但是该`GET`请求是由`XMLHttpRequest`发出的，由于我们测试的时候并没有服务器接收我们的数据，所以我们不能真的发出一个 Http 请求，那么我们如何才能避免真的发出请求呢？又如何能得到返回的数据呢？
别着急，这个时候就到了 Sinon 出场了。我们一开始在就在`test.html`中引用了 Sinon 的库，有了 Sinon，我们就可以模拟服务器响应。我们可以将XMLHttpRequest用一个替身来代替，我们称之为 fake XMLHttpRequest，这样我们就能在测试中轻松控制Ajax请求了。
我们需要稍微更新一下我们的测试用例框架，以下是更改之后的`test.js`文件。

```
chai.should();

describe('TestAPI', function() {
    beforeEach(function() {
        this.xhr = sinon.useFakeXMLHttpRequest();

        this.requests = [];
        this.xhr.onCreate = function(xhr) {
            this.requests.push(xhr);
        }.bind(this);
    });

    afterEach(function() {
        this.xhr.restore();
    });


    //Tests etc. go here
});
```

`beforeEach`和`afterEach`就像他们的名字一样，将会在每一次的测试前和测试后被调用。在每一个测试之前，我们实例化了一个 fake XMLHttpRequest 并且将每一个被创建的 fake request 放入了一个数组中，这些值被存储在`this.xhr`和`this.request`中，这样我们就可以在该测试中的其他函数中使用了。

在每一次测试之后，我们使用了`this.xhr.restore`来恢复初始的XMLHttpRequest对象。

现在，我们可以开始在`test.js`中写下我们的第一个测试集：

```
it('should parse fetched data as JSON', function(done) {
    var data = { foo: 'bar' };
    var dataJson = JSON.stringify(data);

    testApi.get(function(err, result) {
        result.should.deep.equal(data);
        done();
    });

    this.requests[0].respond(200, { 'Content-Type': 'text/json' }, dataJson);
});
```

我们定义了一个对象`data`和它的JSON版本`dataJson`作为我们将要传递的数据。下一步，我们调用了`testApi.call`,在它的回掉函数中我们使用了`result.should.deep.equal`来验证结果是不是与我们所期望的数据一致。我们还调用了`done()`，它的作用是告诉 Mocha 该异步测试完成了。在这里，要注意`done`是测试函数中的一个参数。
最后，我们调用了`this.requests[0].respond`。你还记得之前的`beforeEach`函数么？它在每一次的测试开始之前，将所有的 fake XMLHttpRquest 放入了`this.requests`中。当我们的测试调用`testApi.get`时，它创建了一个请求。这个请求被存入了`this.requests`这个数组中，`this.requests[0]`就代表了这个请求。
通常来说，XMLHttpRequest 没有`respond`函数，这里的`respond`用来响应一个 fake request。我们在该响应中设置状态码为`200`，意思是成功响应。同时，我们将响应头中的`Content-Type`设置为`text/json`，这是因为我们传递的是 JSON 数据。`respond`函数中的最后一个参数表示响应体，我们将其设置为之前创建好的`dataJson`变量。
fake XMLHttpRequest 模拟了一个 GET 请求的响应，在得到响应之后，`testApi.get`中的回掉函数将会被调用。该回调函数中，我们将响应中得到的结果与`data`变量做对比：

```
testApi.get(function(err, result) {
    result.should.deep.equal(data);
    done();
});
```

因为之前我们创建了`data`和`dataJson`变量来代表传递的数据，所以如果响应正确，响应的数据应该被解析成一个对象。
现在，我们可以在浏览器中运行我们的测试了，打开`test.html`文件，你应该可以看到关于测试通过的信息。

##测试POST请求
在我们的示例中还有一个向服务器发送数据的`post`函数，发送的数据是 JSON 格式的。接下来我们就要完成对该函数的测试。

```
it('should send given data as JSON body', function() {
    var data = { hello: 'world' };
    var dataJson = JSON.stringify(data);

    testApi.post(data, function() { });

    this.requests[0].requestBody.should.equal(dataJson);
});
```

就和之前一样，我们首先定义了一个测试数据`data`和它的 JSON 格式的变量`dataJson`。之后我们调用了`testApi.post`。与之前测试 GET 请求不一样的地方是，这一次我们只需要验证要被发送的数据是否被正确的转换成了 JSON 格式，因为我们只需要保证 POST 请求中发出的数据是正确的，因此我们的回掉函数是空的。
该段代码中最后一行使用了一个断言来确定发送数据的正确性。与之前一样，我们使用了 fake XMLHttpRequest，但是这一次我们要证明它携带了正确的数据。POST 请求中携带的数据存放在 fake XMLHttpRequest 的 `requestBody`属性中，我们将其与`dataJson`作比较来验证我们的行为。

##错误测试
作为最后一个示例，让我们来是测试一个失败的请求。因为网络连接中可能出现很多问题，同时服务器也可能出现问题，并且我们不应该让我们网站的用户对具体的错误信息感到疑惑，所以错误测试非常重要。
示例模块中的回调函数中有两个参数，第一个参数就是错误信息，第二个参数则是每一次 Http 请求响应得到的结果。代码如下：

```
it('should return error into callback', function(done) {
    testApi.get(function(err, result) {
        err.should.exist;
        done();
    });

    this.requests[0].respond(500);
});
```

由于是错误测试，所以这一次我们不需要任何数据。我们调用了`testApi.get`，并在其回掉函数中来验证 error 参数的存在。为了模拟响应错误，最后一行中的 fake XMLHttpRequest 发送了一个内部服务器错误的状态码 500 来触发错误处理程序。

##总结
Ajax请求的测试是很重要的，如果你能证明每一次请求都是正确的，那么你应用程序中的其他部分就能完全相信每一次 Ajax 请求得到的数据。
假如你正在使用 JQuery Ajax，测试的方法与例子的方法是一模一样的。你同样可以使用 Sinon 中的 fake XMLHttpRequests。
当然，Sinon 中并不只有 fake XMLHttpRequest， 它还有 fake Server 用来模拟服务器响应，感兴趣的话可以去 Sinon 的官网了解。