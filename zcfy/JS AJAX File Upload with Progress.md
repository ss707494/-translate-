# 带进度条的ajax文件上传 (JS AJAX File Upload with Progress)
原文链接:[http://codular.com/javascript-ajax-file-upload-with-progress](http://codular.com/javascript-ajax-file-upload-with-progress)

现在,人们喜欢在浏览网页时做一些其他事情而不离开该网页,这通常是通过ajax来实现.大多数情况,人们使用jQuery来实现,但是随着浏览器的进步,人们比不需要这么做.这里我们将介绍如何在不离开页面的情况下将文件上传到服务器，我们将使用与我们[之前的文章](http://codular.com/php-file-uploads)中使用的相同的后端PHP代码.
该脚本将上传文件至服务器,同时显示上传进度,并最终返回上传文件的链接地址.在某些情况下,你可能想要返回上传文件的id或者其他的应用信息.
**Note:** 该代码不支持较老的ie浏览器,通过[Can I use](http://caniuse.com/xhr2)我们只支持ie10+

## Let's Code

我们将从HTML结构开始，然后是JavaScript，然后我将给您提供PHP代码，这部分改编于之前教程 - 对PHP代码将不会有太多的解释。

### HTML

我们只需要使用两个输入框，一个是文件类型`file`，另一个只是一个按钮`button`,以便我们可以监听到它被点击发送文件上传请求。 我们还将有一个div，我们改变宽度以突出显示上传的状态。

如下所示:


```
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>JS File Upload with Progress</title>
    <style>
    .container {
        width: 500px;
        margin: 0 auto;
    }
    .progress_outer {
        border: 1px solid #000;
    }
    .progress {
        width: 20%;
        background: #DEDEDE;
        height: 20px;  
    }
    </style>
</head>
<body>
    <div class='container'>
        <p>
            Select File: <input type='file' id='_file'> <input type='button' id='_submit' value='Upload!'>
        </p>
        <div class='progress_outer'>
            <div id='_progress' class='progress'></div>
        </div>
    </div>
    <script src='upload.js'></script>
</body>
</html>

```
你将看到我们写了一点进度条样式,并在底部加入脚本文件来处理文件上传以及进度条展示.

### JavaScript

首先, 我们需要拿到我们将要使用的标签,他们已经用id标记上.

```
var _submit = document.getElementById('_submit'), 
_file = document.getElementById('_file'), 
_progress = document.getElementById('_progress');

```

下一步,给`_submit`添加点击事件,用以上传我们选择的文件.为此,我们将使用`addEventListener`方法,点击按钮后让其调用`upload`方法.

```
`_submit.addEventListener('click', upload);`

```
现在我们可以继续处理上传,有以下几步:
1. 检查被选中的文件
2. 动态创建要发送的文件数据
3. 通过js创建XMLHttpRequest
4. 上传文件

#### 检查被选中的文件

我们的文件输入框`_file`有一个查询被选中文件队列的参数`files`-如果你设置了`multiple`参数将可以选多个文件.我们做简单的检查判断,如果该数组长度大于0,则继续,否则直接返回.

```
if(_file.files.length === 0){
    return;
}

```
现在,我们能确保已选择一个文件,我们将假定有一个文件,请记着数组的索引以0开头.


#### 动态创建要发送的文件数据

为此,我们需要使用`FormData`,并将数据加入其中.下一步,我们可以在第3步生成的request中发送我们的`FormData`.我们使用的`append`方法,第一个参数与输入框的`name`属性相似,第二个参数是值`value`.
这里,我们将`value`设为我们选择的第一个文件.

```
var data = new FormData();
data.append('SelectedFile', _file.files[0]);

```

当稍后向服务端发送数据时,我们将使用它.

#### 通过上传脚本创建XMLHttpRequest

这部分是非常基础的，我们将创建一个新的`XMLHttpRequest`，并设置一些设置。首先我们将修改`onreadystatechange`的值来定义请求状态改变时的回调函数.该方法将会在状态改变时检查`readyState`,确保该值是我们想要的-在这个例子中就是4,代表请求完成.

第二步,我们将在`upload`属性上添加`progress`事件.这样我们能得到上传进度用来更新进度条.

```
var request = new XMLHttpRequest();
request.onreadystatechange = function(){
    if(request.readyState == 4){
        try {
            var resp = JSON.parse(request.response);
        } catch (e){
            var resp = {
                status: 'error',
                data: 'Unknown error occurred: [' + request.responseText + ']'
            };
        }
        console.log(resp.status + ': ' + resp.data);
    }
};

```
当请求成功后,我们用`try ... catch`包裹解析返回值的过程,若解析失败,我们将创建我们自己的返回对象来使得后面的代码能不报错.可以自行决定如何处理返回值,这里我们只是将其输出至控制台.

现在我们来处理进度条:

```
request.upload.addEventListener('progress', function(e){
    _progress.style.width = Math.ceil(e.loaded/e.total) * 100 + '%';
}, false);

```

这里有一点点复杂,我们监听一个事件,该事件对象有两个我们比较关注的属性,`loaded`和`total`.`loaded`表示已经上传到服务器端的数值,`total`表示要发送的总数值,我们可以根据这两个值计算一个百分比,来设置进度条的宽度.

_Note:_ 这里没有加入任何动画特效,但你可以根据需要自定义动画效果.

#### 上传文件

现在我们可以发送请求，我们将通过POST请求到一个名为`upload.php`的文件，并使用`send（）`方法，参数为`data`，这样我们就可以发送数据:
```
request.open('POST', 'upload.php');
request.send(data);

```
下面给出完整的JavaScript代码:


```
var _submit = document.getElementById('_submit'), 
_file = document.getElementById('_file'), 
_progress = document.getElementById('_progress');

var upload = function(){

    if(_file.files.length === 0){
        return;
    }

    var data = new FormData();
    data.append('SelectedFile', _file.files[0]);

    var request = new XMLHttpRequest();
    request.onreadystatechange = function(){
        if(request.readyState == 4){
            try {
                var resp = JSON.parse(request.response);
            } catch (e){
                var resp = {
                    status: 'error',
                    data: 'Unknown error occurred: [' + request.responseText + ']'
                };
            }
            console.log(resp.status + ': ' + resp.data);
        }
    };

    request.upload.addEventListener('progress', function(e){
        _progress.style.width = Math.ceil(e.loaded/e.total) * 100 + '%';
    }, false);

    request.open('POST', 'upload.php');
    request.send(data);
}

_submit.addEventListener('click', upload);

```

现在到了PHP...

### PHP
这是我们使用的代码,你将注意到一些区别,主要是我们用最上面的JSON方法来返回值都作为JSON格式输出.这个PHP与之前文章中的代码相同,这也就意味着该方法只适用于小于500Kb的PNG图片.此外,成功信息将返回已上传文件的路径:

```
<?php
// Output JSON
function outputJSON($msg, $status = 'error'){
    header('Content-Type: application/json');
    die(json_encode(array(
        'data' => $msg,
        'status' => $status
    )));
}

// Check for errors
if($_FILES['SelectedFile']['error'] > 0){
    outputJSON('An error ocurred when uploading.');
}

if(!getimagesize($_FILES['SelectedFile']['tmp_name'])){
    outputJSON('Please ensure you are uploading an image.');
}

// Check filetype
if($_FILES['SelectedFile']['type'] != 'image/png'){
    outputJSON('Unsupported filetype uploaded.');
}

// Check filesize
if($_FILES['SelectedFile']['size'] > 500000){
    outputJSON('File uploaded exceeds maximum upload size.');
}

// Check if the file exists
if(file_exists('upload/' . $_FILES['SelectedFile']['name'])){
    outputJSON('File with that name already exists.');
}

// Upload file
if(!move_uploaded_file($_FILES['SelectedFile']['tmp_name'], 'upload/' . $_FILES['SelectedFile']['name'])){
    outputJSON('Error uploading file - check destination is writeable.');
}

// Success!
outputJSON('File uploaded successfully to "' . 'upload/' . $_FILES['SelectedFile']['name'] . '".', 'success');

```
如果将所有内容都放在一起，您应该可以将文件上传到您期望的位置，并在浏览器的控制台内成功返回。


## 结束语
还有一些比较容易而又有效的方式来增强用户体验.通过将文件队列的多个文件加入到`FormData`,可以实现多文件上传.
有一点要说明,如果你是在本地做测试,你可能没办法看到进度条逐步变化,这取决于你本地机器的上传速度,我建议在服务器上使用较大的png文件做测试.
                