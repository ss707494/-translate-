# Javascript中的Promise实践 - Promise链式调用(Practical Promises in JavaScript - Returning New Promises with Chaining Promises)

欢迎来到我的Promise教程系列的第四部分!在[第一部分](http://trycatchfail.com/blog/post/Practical-Promises-in-JavaScript-What-are-they-and-how-do-I-use-them)我们讲到promise是什么和我们能用它做什么.在[第二部分](http://trycatchfail.com/blog/post/Practical-Promises-in-JavaScript-Making-Promises)我们学习了怎样创建promise.然后[第三部分](http://trycatchfail.com/blog/post/Practical-Promises-in-JavaScript-The-Basics-of-Promise-Chaining)我们知道了为何每次调用`then`方法都可以生成一个promise,这样这些promise就可以实现链式调用.在本章中,我们将学习如何应用这些来简化我们的异步代码.

你是否也曾看过(或写过)这样的代码?

```
widgetSvc.getWidget(id).then(function(widget) {
    $ctrl.widget = widget;
    listingSvc.getProductListing(widget.listingId).then(function(listing) {
        $ctrl.price = listing.price;

        manufacturerSvc.getManufacturer(listing.manufacturer.Id).then(function(manufacturer) {
            $ctrl.manufacturerName = manufacturer.name;
        });
    })
})

```

这就是一段深度嵌套的代码:有三个异步函数通过函数嵌套连接到一起.而且在这些年里,我在很多实际项目都看到过这样类似的代码.过去没有找到较好的方式来处理这个.伙计,这就是"回调地狱".

# 更好的方法

幸运的是,promise给我们提供了一个较好的办法来解决这个问题!在[上一部分](http://trycatchfail.com/blog/post/Practical-Promises-in-JavaScript-The-Basics-of-Promise-Chaining)中,我们学会了**链式调用**`then`方法,因为无论`then`方法里面返回什么最终都会将结果处理成一个promise.

我们可以依赖同一种方式,每次返回一个**新的**需要解决(resolved)的promise,然后就可以**链式调用**这些promise.让我们来看一个简单的例子!

我们首先创建一个promise来处理一个异步函数...

```
const promise = new Promise(resolve => setTimeout(() => resolve(1), 2000));

```

然后我们在他的`then`方法中返回**另一个**新的promise...

```
const childPromise = promise.then(result => {
    console.log(`First callback - ${new Date().getSeconds()}s: ${result}`);
    return new Promise(resolve => setTimeout(() => resolve(result+1), 2000));
});

```

然后在这个子promise的`then`方法中**也**返回一个新的promise...

```
const grandchildPromise = childPromise.then(result => {
    console.log(`Second callback - ${new Date().getSeconds()}s: ${result}`);
    return new Promise(resolve => setTimeout(() => resolve(result+1), 2000));
});

```

为了更好的测试,我们再添加最后一个`then`回调方法...

```
grandchildPromise.then(result => {
    console.log(`Third callback - ${new Date().getSeconds()}s: ${result}`);
});

```

这个链式的promise将会产生如下输出:

```
First callback - 24s: 1
Second callback - 26s: 2
Third callback - 28s: 3

```

上面的代码显得比较冗长.我们可以重构一下让他更加简洁:

```
const promise = new Promise(resolve => setTimeout(() => resolve(1), 2000));
promise.then(result => {
    console.log(`First callback - ${new Date().getSeconds()}s: ${result}`);
    return new Promise(resolve => setTimeout(() => resolve(result+1), 2000));
}).then(result => {
    console.log(`Second callback - ${new Date().getSeconds()}s: ${result}`);
    return new Promise(resolve => setTimeout(() => resolve(result+1), 2000));
}).then(result => {
    console.log(`Third callback - ${new Date().getSeconds()}s: ${result}`);
});

```

他将会有相同的输出!

# 总结一下

现在我们已经了解了如何链式地使用promise,并知道了他的原理,这样我们就可以轻松的重构最上面的那段代码:

```
widgetSvc.getWidget(id).then(widget => {
    $ctrl.widget = widget;
    return listingSvc.getProductListing(widget.listingId);
}).then(listing => {
    $ctrl.price = listing.price;
    return manufacturerSvc.getManufacturer(listing.manufacturer.Id);
}).then(manufacturer => {
    $ctrl.manufacturerName = manufacturer.name;
});

```

我们需要做的就是在每一个`then`方法中返回 _下一个_ promise.下一个`then`被绑定到我们返回的promise上,这样就可以保证在上一个异步函数执行完毕后下一个才开始执行!

请记住,我们依然可以使用之前学到的关于promise的知识!我们可以在一个`then`回调方法里建立 _多个_ promise异步方法,只需要将他们用`Promise.all`组合到一起,就像这样:

```
widgetSvc.getWidget(id).then(widget => {
    $ctrl.widget = widget;
    return listingSvc.getProductListing(widget.listingId);
}).then(listing => {
    $ctrl.price = listing.price;
    return Promise.all([
        manufacturerSvc.getManufacturer(listing.manufacturer.Id),
        resellerSvc.getReseller(listing.reseller.Id)
    ]);
}).then((manufacturer, reseller) => {
    $ctrl.manufacturerName = manufacturer.name;
    $ctrl.resellerName = reseller.name;
});

```

以上就是本章的"Promise实践"教程,但是我们还没有完结!在下一章里,我们将学习通过Promise _展开_ 一些东西,以及如何利用这一概念在javascript中构建清晰的api接口.
                