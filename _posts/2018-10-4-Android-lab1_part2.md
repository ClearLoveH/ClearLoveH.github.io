---
layot:     post
title:      "中山大学智慧健康服务平台应用开发"
subtitle:   "基础的事件处理"
date:       2018-10-4 16:24
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Android
---

>国庆快乐，身体安康。

---

[项目仓库](https://gitee.com/ClearLoveH/PersonalProject1)

---

## 一、实验题目 

### 基础的事件处理

---

## 二、实现内容

实现一个Android应用，界面呈现如图中的效果。
  
![](/img/in-post/post-Android/lab1/截图/preview.jpg)

要求:

- 该界面为应用启动后看到的第一个界面。
- 各控件处理的要求
    - 点击搜索按钮：
        - 如果搜索内容为空，弹出Toast信息“搜索内容不能为空”。
        - 如果搜索内容为“Health”，根据选中的RadioButton项弹出如下对话框。

            ![](/img/in-post/post-Android/lab1/截图/lab1_week5_要求1.jpg)

        - 点击“确定”，弹出Toast信息——对话框“确定”按钮被点击。
        - 点击“取消”，弹出Toast 信息——对话框“取消”按钮被点击。
        - 否则弹出如下对话框，对话框点击效果同上。

            ![](/img/in-post/post-Android/lab1/截图/lab1_week5_要求2.jpg)

    - RadioButton选择项切换：选择项切换之后，弹出Toast信息“XX被选中”，例如从图片切换到视频，弹出Toast信息“视频被选中”

---

## 三、课堂实验结果
### (1)实验截图

- 点击搜索按钮
    - 如果搜索内容为空，弹出Toast信息“搜索内容不能为空”。

        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_0.png)

    - 输入为Health，搜索成功，根据RadioButton显示对应弹出框的内容

        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_1_1.png)
        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_1_2.png)
        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_1_3.png)
        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_1_4.png)

    - 输入为其他，搜索失败

        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_4.png)

    - 点击对话框的取消按钮

        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_2.png)

    - 点击对话框的确定按钮

        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_3.png)

- 切换RadioButton的选择项
    
    - 选择`图片`
    
        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_5.png)

    - 选择`视频`

        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_6.png)

    - 选择`问答`

        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_7.png)

    - 选择`咨询`

        ![](/img/in-post/post-Android/lab1/截图/lab1_week5_8.png)

### (2)实验步骤以及关键代码


- 给搜索按钮添加需求事件

    给搜索按钮添加一个事件监听器`setOnClickListener`，在点击按钮时触发的事件都在这个监听器中编写即可。

    1. 如果搜索内容为空，弹出Toast信息“搜索内容不能为空”。
        我们怎么判断输入是否为空呢？———直接获取edittext的text文本，并使用`.toString()`函数进行转换，然后判断获取的string长度是否为0即可。
            
        实现这个效果在监听器中添加一个条件判断如下：
        ```java
            if(editString.length()==0){
                    toast= Toast.makeText(getApplicationContext(),"搜索内容不能为空", Toast.LENGTH_SHORT);
                    toast.show();
            }
        ```
    2. 弹出框的添加：
        
        AS中弹出框的组件是` AlertDialog.Builder `，我们通过设置Builder的`Title`、`Message`、`PositiveButton`、`NegativeButton`属性来分别设置弹出框的标题，还是弹出信息，以及两个按钮的文本，操作简单易上手，具体实现如下。

        ```java
        AlertDialog.Builder builder=new AlertDialog.Builder(this);
        builder.setTitle("提示");
        builder.setMessage(healthText);
        builder.setPositiveButton("确定",
                ...
                ...
                ...
                });
        builder.setNegativeButton("取消",
                ...
                ...
                ...
                });
        AlertDialog dialog=builder.create();
        dialog.show();
        ```
    3. 如何判断搜索成功还是失败？又如何实现对应的点击事件？
        - 判断搜索成功还是失败我们都在按钮的监听器中实现，我于此是添加了两个条件判断如下：
            ```java
            if("Health".equals(editString))
                showDialogS(healthText);
            else showDialogF();
            ```
            注意：调用java中String类型的`.equal()`函数来判断输入字符串是否为Health，而不能直接使用 == ，这在java中和C还是有些不同。
        - `showDialogS`、`showDialogF`是我额外定义在外部的事件函数。分别实现搜索成功和失败的触发效果。
            - `showDialogF()`为实现搜索失败时触发的对应弹框效果

            ```java
            private void showDialogF(){
                AlertDialog.Builder builder=new AlertDialog.Builder(this);
                builder.setTitle("提示");
                builder.setMessage("搜索失败");
                builder.setPositiveButton("确定",
                        ...
                        ...
                        ...
                        });
                builder.setNegativeButton("取消",
                        ...
                        ...
                        ...
                    });
                AlertDialog dialog=builder.create();
                dialog.show();

            }
            ```
            - `showDialogS(String healthText)`为实现搜索成功时的弹框效果，内容基本与搜索失败时的设置无差，但是我们注意到我这里添加了一个传入的String类型的参数，这是为什么呢？
                - 这个参数的设置是因为我们需要根据选中的RadioButton项调整搜索成功时的弹出框内容，于是我们在整个java文件activity类中声名一个私有变量`healthText`，目的是获取选中的RadioButton的信息，具体便是我们在后面添加RadioButton的toast时，顺便对healthText进行修改，然后再通过一个参数传进给`showDialogS()`函数，最后通过
                    ```java
                    builder.setMessage(healthText);
                    ```
                    来最终实现弹出框随选中不同的RadioButton来显示对应的内容。

            - 点击“确定”，与点击“取消”按钮时都要弹出对应的toast，这里在`showDialogS`、`showDialogF`的builder中的`PositiveButton`和`NegativeButton`中都对应添加一个监听器，然后将toast的显示事件添加进去即可，取消按钮如下：
                ```java
                builder.setNegativeButton("取消",
                            new DialogInterface.OnClickListener() {
                            public void onClick(DialogInterface dialogInterface, int i) {
                                toast= Toast.makeText(getApplicationContext(),"对话框“取消”按钮被点击", Toast.LENGTH_SHORT);
                                toast.show();
                            }
                        });
                ```
                确认按钮也同样添加。

- 给RadioGroup中选中不同的RadioButton添加选择事件。
    - 搜索按钮有他的点击事件的监听器，选择按钮也同样有他的选择事件监听器，便是`setOnCheckedChangeListener`，监听器有自带的参数是`(group,checkId)`，group便是指明进行选择操作的当前小组的ID，checkId则是指明选择了哪一个按钮，我们只需要获取checkId，然后使用switch条件语句，分别添加对应的toast即可。
        ```java
        selectButton.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
                @Override
                public void onCheckedChanged(RadioGroup group, int checkedId) {
                    switch (checkedId) {
                        case R.id.radioButton1:
                            toast= Toast.makeText(getApplicationContext(),"图片被选中", Toast.LENGTH_SHORT);
                            healthText="图片搜索成功";
                            toast.show();
                            break;
                        case R.id.radioButton2:
                            toast= Toast.makeText(getApplicationContext(),"视频被选中", Toast.LENGTH_SHORT);
                            healthText="视频搜索成功";
                            toast.show();
                            break;
                        case R.id.radioButton3:
                            toast= Toast.makeText(getApplicationContext(),"问答被选中", Toast.LENGTH_SHORT);
                            healthText="问答搜索成功";
                            toast.show();
                            break;
                        case R.id.radioButton4:
                            toast= Toast.makeText(getApplicationContext(),"资讯被选中", Toast.LENGTH_SHORT);
                            healthText="资讯搜索成功";
                            toast.show();
                            break;
                    }
                }
            });
        ```
    - 注意前一部分中我们需要获取选中按钮的信息，所以我们在这里同时添加对healthText的修改，来达到反馈选中按钮信息的目的，而且我们默认要求是事先选中第一个图片按钮，所以我们的healthText初始化为`图片搜索成功`。
    
这便是我们这次实验的全部实验步骤。


### (3)实验遇到的困难以及解决思路

这次基本事件处理的实验遇到的主要问题有两个：

1. 搜索成功时弹出框的内容如何需要根据RadioButton选中的内容来改变
    - 这一部分的内容是在做完其余的内容后，和其他同学讨论时才发现自己忽略掉了，发现漏了这部分的问题之后呢，我就在思考怎么获取RadioGroup的选中信息，于是决定使用一个局部变量healthText来进行搜索框和选择按钮的一个“沟通”，大家都使用healthText来作为中间变量，选择按钮把信息传递给它，然后搜索框在调用它从而实现弹出框的内容随选择按钮改变而改变。

2. 为什么判断输入框的text为`Health`总是显示搜索失败？
    - 一开始由于c语言学习的一个惯性思维，在判断输入框的text的条件语句中，我使用连等号来进行判断，却发现不论我输入内容是什么，最终都是搜索失败。经过debug后发现，是问题出现在==这里，同时我也学习到了一个java的知识点————`事实上，java用“==”来判断字符串变量是否相等时，它是通过判断两个变量的标识（也就是id号）来决定的，也就是判断变量是否指向同一引用对象，即如果a和b同时指向的是内存中的同一个对象，那么他们的id号是相同的，内容当然也是相同的，而我们这里要判断的是两个不同对象中的字符串内容是否相等，很明显，是不同的对象，那么，用“==”来判断的话，结果肯定都是false了。`所以我们这里改为使用string类型的.equal()函数来判断是否相等即可。

---

## 四、实验思考及感想

实验中的思考与感悟：

- 这次任务是安卓的事件处理的基础，与第一次UI任务不同，这次的任务涉及的层次更加深一些，做完这次任务之后，我对事件监听器的了解更加的深刻，使用起来也是非常的顺手，对`Buileder`、`toast`等组件的使用也明白清楚。

- 这是我们学习安卓的第一个项目的第二个部分，在学习AS中基本组件的使用的同时，也帮助我发现了许多在java上曾经理解出错的误区，对java语言的掌握也更深一层；同时我对不同部件间的沟通问题也有了组件想法，对AS的编程架构也逐渐明晰了许多，UI与基本事件的分离让我们编程人员的思路更加的清晰。这次作业依旧是国庆前就已经做完，希望后面可以保持这样的状态继续下去。 

---

