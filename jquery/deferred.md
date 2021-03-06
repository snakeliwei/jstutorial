---
title: jQuery.Deferred对象
category: jquery
date: 2012-12-07
layout: page
modifiedOn: 2013-08-25
---

## 概述

deferred对象是jQuery对Promises接口的实现。它是非同步操作的通用接口，可以被看作是一个等待完成的任务，开发者通过一些通过的接口对其进行设置。事实上，它扮演代理人（proxy）的角色，将那些非同步操作包装成具有某些统一特性的对象，典型例子就是Ajax操作、网页动画、web worker等等。

jQuery的所有Ajax操作函数，默认返回的就是一个deferred对象。

## Promises是什么

由于JavaScript单线程的特点，如果某个操作耗时很长，其他操作就必需排队等待。为了避免整个程序失去响应，通常的解决方法是将那些排在后面的操作，写成“回调函数”（callback）的形式。这样做虽然可以解决问题，但是有一些显著缺点：

- 回调函数往往写成函数参数的形式，导致函数的输入和输出非常混乱，整个程序的可阅读性差；
- 回调函数往往只能指定一个，如果有多个操作，就需要改写回调函数。
- 整个程序的运行流程被打乱，除错和调试的难度都相应增加。

Promises就是为了解决这些问题而提出的，它的主要目的就是取代回调函数，成为非同步操作的解决方案。它的核心思想就是让非同步操作返回一个对象，其他操作都针对这个对象来完成。比如，假定ajax操作返回一个Promise对象。

{% highlight javascript %}

var promise = get('http://www.example.com');

{% endhighlight %}

然后，Promise对象有一个then方法，可以用来指定回调函数。一旦非同步操作完成，就调用指定的回调函数。

{% highlight javascript %}

promise.then(function (content) {
  console.log(content)
})

{% endhighlight %}

可以将上面两段代码合并起来，这样程序的流程看得更清楚。

{% highlight javascript %}

get('http://www.example.com').then(function (content) {
  console.log(content)
})

{% endhighlight %}

在1.7版之前，jQuery的Ajax操作采用回调函数。

{% highlight javascript %}

$.ajax({
    url:"/echo/json/",
    success: function(response)
    {
       console.info(response.name);
    }
});

{% endhighlight %}

1.7版之后，Ajax操作直接返回Promise对象，这意味着可以用then方法指定回调函数。

{% highlight javascript %}

$.ajax({
    url: "/echo/json/",
}).then(function (response) {
    console.info(response.name);
});

{% endhighlight %}

## deferred对象的方法

### $.deferred()方法

作用是生成一个deferred对象。

{% highlight javascript %}

var deferred = $.deferred();

{% endhighlight %}

### done() 和 fail() 

这两个方法都用来绑定回调函数。done()指定非同步操作成功后的回调函数，fail()指定失败后的回调函数。

{% highlight javascript %}

var deferred = $.Deferred();

deferred.done(function(value) {
   alert(value);
});

{% endhighlight %}

它们返回的是原有的deferred对象，因此可以采用链式写法，在后面再链接别的方法（包括done和fail在内）。

### resolve() 和 reject()

这两个方法用来改变deferred对象的状态。resolve()将状态改为非同步操作成功，reject()改为操作失败。

{% highlight javascript %}

var deferred = $.Deferred();

deferred.done(function(value) {
   alert(value);
});

deferred.resolve("hello world");

{% endhighlight %}

一旦调用resolve()，就会依次执行done()和then()方法指定的回调函数；一旦调用reject()，就会依次执行fail()和then()方法指定的回调函数。

### state方法

该方法用来返回deferred对象目前的状态。

{% highlight javascript %}

var deferred = new $.Deferred();
deferred.state();  // "pending"
deferred.resolve();
deferred.state();  // "resolved"

{% endhighlight %}

该方法的返回值有三个：

- pending：表示操作还没有完成。
- resolved：表示操作成功。
- rejected：表示操作失败。

### notify() 和 progress()

progress()用来指定一个回调函数，当调用notify()方法时，该回调函数将执行。它的用意是提供一个接口，使得在非同步操作执行过程中，可以执行某些操作，比如定期返回进度条的进度。

{% highlight javascript %}

	var userProgress = $.Deferred();
    var $profileFields = $("input");
    var totalFields = $profileFields.length
        
    userProgress.progress(function (filledFields) {
        var pctComplete = (filledFields/totalFields)*100;
        $("#progress").html(pctComplete.toFixed(0));
    }); 

    userProgress.done(function () {
        $("#thanks").html("Thanks for completing your profile!").show();
    });
    
    $("input").on("change", function () {
        var filledFields = $profileFields.filter("[value!='']").length;
        userProgress.notify(filledFields);
        if (filledFields == totalFields) {
            userProgress.resolve();
        }
    });

{% endhighlight %}

### then()

then()的作用也是指定回调函数，它可以接受三个参数，也就是三个回调函数。第一个参数是resolve时调用的回调函数，第二个参数是reject时调用的回调函数，第三个参数是progress()方法调用的回调函数。

{% highlight javascript %}

deferred.then( doneFilter [, failFilter ] [, progressFilter ] )

{% endhighlight %}

在jQuery 1.8之前，then()只是.done().fail()写法的语法糖，两种写法是等价的。在jQuery 1.8之后，then()返回一个新的deferred对象，而done()返回的是原有的deferred对象。如果then()指定的回调函数有返回值，该返回值会作为参数，传入后面的回调函数。

{% highlight javascript %}

var defer = jQuery.Deferred();

defer.done(function(a,b){
            return a * b;
}).done(function( result ) {
            console.log("result = " + result);
}).then(function( a, b ) {
            return a * b;
}).done(function( result ) {
            console.log("result = " + result);
}).then(function( a, b ) {
            return a * b;
}).done(function( result ) {
            console.log("result = " + result);
});

defer.resolve( 2, 3 );

{% endhighlight %}

在jQuery 1.8版本之前，上面代码的结果是：

{% highlight javascript %}

result = 2 
result = 2 
result = 2 

{% endhighlight %}

在jQuery 1.8版本之后，返回结果是

{% highlight javascript %}

result = 2 
result = 6 
result = NaN 

{% endhighlight %}

这一点需要特别引起注意。

{% highlight javascript %}

$.ajax( url1, { dataType: "json" } )
.then(function( data ) {
    return $.ajax( url2, { data: { user: data.userId } } );
}).done(function( data ) {
  // 从url2获取的数据
});

{% endhighlight %}

上面代码最后那个done方法，处理的是从url2获取的数据，而不是从url1获取的数据。

利用then()会修改返回值这个特性，我们可以在调用其他回调函数之前，对前一步操作返回的值进行处理。

{% highlight javascript %}

var post = $.post("/echo/json/")
	.then(function(p){
        return p.firstName;
	});

post.done(function(r){ console.log(r); });

{% endhighlight %}

上面代码先使用then()方法，从返回的数据中取出所需要的字段（firstName），所以后面的操作就可以只处理这个字段了。

有时，Ajax操作返回json字符串里面有一个error属性，表示发生错误。这个时候，传统的方法只能是通过done()来判断是否发生错误。通过then()方法，可以让deferred对象调用fail()方法。

{% highlight javascript %}

var myDeferred = $.post('/echo/json/', {json:JSON.stringify({'error':true})})
    .then(function (response) {
            if (response.error) {
                return $.Deferred().reject(response);
            }
            return response;
        },function () {
            return $.Deferred().reject({error:true});
        }
    );

myDeferred.done(function (response) {
        $("#status").html("Success!");
    }).fail(function (response) {
        $("#status").html("An error occurred");
    });

{% endhighlight %}

### always()

always()也是指定回调函数，不管是resolve或reject都要调用。

## promise对象

大多数情况下，我们不想让用户从外部更改deferred对象的状态。这时，你可以在deferred对象的基础上，返回一个针对它的promise对象。我们可以把后者理解成，promise是deferred的只读版，或者更通俗地理解成promise是一个对将要完成的任务的承诺。

你可以通过promise对象，为原始的deferred对象添加回调函数，查询它的状态，但是无法改变它的状态，也就是说promise对象不允许你调用resolve和reject方法。

{% highlight javascript %}

function getPromise(){
    return $.Deferred().promise();
}

try{
    getPromise().resolve("a");
} catch(err) {
    console.log(err);
}

{% endhighlight %}

上面的代码会出错，显示TypeError {} 。

jQuery的ajax() 方法返回的就是一个promise对象。此外，Animation类操作也可以使用promise对象。

{% highlight javascript %}

var promise = $('div.alert').fadeIn().promise();

{% endhighlight %}

## $.when()方法

$.when()接受多个deferred对象作为参数，当它们全部运行成功后，才调用resolved状态的回调函数，但只要其中有一个失败，就调用rejected状态的回调函数。它相当于将多个非同步操作，合并成一个。

{% highlight javascript %}

$.when(
    $.ajax( "/main.php" ),
    $.ajax( "/modules.php" ),
    $.ajax( "/lists.php" )
).then( successFunc, failureFunc );

{% endhighlight %}

## 实例

### setTimeout()的改写

我们可以用deferred对象改写setTimeout()

{% highlight javascript %}

$.wait = function(time) {
  return $.Deferred(function(dfd) {
    setTimeout(dfd.resolve, time);
  });
}

{% endhighlight %}

使用方法如下：

{% highlight javascript %}

$.wait(5000).then(function() {
  alert("Hello from the future!");
});

{% endhighlight %}

### 自定义操作使用deferred接口

我们可以利用deferred接口，使得任意操作都可以用done()和fail()指定回调函数。

{% highlight javascript %}

Twitter = {
  search:function(query) {
    var dfr = $.Deferred();
    $.ajax({
     url:"http://search.twitter.com/search.json",
     data:{q:query},
     dataType:'jsonp',
     success:dfr.resolve
    });
    return dfr.promise();
  }
}

{% endhighlight %}

使用方法如下：

{% highlight javascript %}

Twitter.search('intridea').then(function(data) {
  alert(data.results[0].text);
});

{% endhighlight %}

deferred对象的另一个优势是可以附加多个回调函数。

{% highlight javascript %}

function doSomething(arg) {
  var dfr = $.Deferred();
  setTimeout(function() {
    dfr.reject("Sorry, something went wrong.");
  });
  return dfr;
}

doSomething("uh oh").done(function() {
  alert("Won't happen, we're erroring here!");
}).fail(function(message) {
  alert(message)
});

{% endhighlight %}

## 参考链接

- [jQuery.Deferred is the most important client-side tool you have](http://eng.wealthfront.com/2012/12/jquerydeferred-is-most-important-client.html)
- [Fun With jQuery Deferred](http://www.intridea.com/blog/2011/2/8/fun-with-jquery-deferred)
- Bryan Klimt, [What’s so great about JavaScript Promises?](http://blog.parse.com/2013/01/29/whats-so-great-about-javascript-promises/)
- José F. Romaniello, [Understanding JQuery.Deferred and Promise](http://joseoncode.com/2011/09/26/a-walkthrough-jquery-deferred-and-promise/)
- Julian Aubourg, Addy Osmani, [Creating Responsive Applications Using jQuery Deferred and Promises](http://msdn.microsoft.com/en-us/magazine/gg723713.aspx)
