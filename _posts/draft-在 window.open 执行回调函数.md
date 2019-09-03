---
layout:     post
title:      在 window.open 执行回调函数
subtitle:   
date:       2019-09-03
author:     Sam
header-img: 
catalog: true
tags:
    - 技术
    - JavaScript
---
### 在 window.open 执行回调函数

#### 1. 主页
```html
<!-- parent.html -->
<script type="text/javascript">
    function Button_OnClick() {
        ShowModal("Message from parent window.", UpdateContent);
    }

    function UpdateContent(msg) {
        document.write(msg);
    }

    function ShowModal(msg, callBack) {

        if (window.showModalDialog) {
            var retValue = showModalDialog("./child.html", msg, "dialogWidth:200px; dialogHeight:200px;");
            if (retValue) {
                UpdateContent(retValue);
            }
        }
        else {
            // we want similar functionality in Opera, but it's not modal!
            var modal = window.open("./child.html", null, "width=200,height=200", null);
            modal.dialogArguments = msg;
            modal.callBack = callBack;
        }
    }
</script>

<body>
    <button onclick="Button_OnClick ()">open dialog</button>
</body>
```

#### 2. 子窗口
```html
<!-- child.html -->
<script type="text/javascript">
    function Init() {
        var msg = window.dialogArguments;
        alert(msg);
    }

    function Execute() {
        var msg = "Message from child window.";

        if (window.showModalDialog) {
            window.returnValue = msg;
        }
        else {
            // window.opener.UpdateContent (msg);
            window.callBack(msg);
        }
    }
</script>

<body onload="Init()" onunload="Execute()">
</body>
```

#### 3. 使用回调函数
通过给 window 对象添加回调函数，进而调用主页的方法。