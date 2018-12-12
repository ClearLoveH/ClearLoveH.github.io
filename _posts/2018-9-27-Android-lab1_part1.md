---
layot:     post
title:      "中山大学智慧健康服务平台应用开发"
subtitle:   "基本的UI界面设计"
date:       2018-9-27 13:39
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Android
---

>中秋快乐，身体安康。

---

[项目仓库](https://gitee.com/ClearLoveH/PersonalProject1)

---
## 一、实验题目 

### 基本的UI界面设计

---

## 二、实现内容

实现题目所要求的UI界面
  
![](/img/in-post/post-Android/lab1/截图/preview.jpg)


各控件的要求:

- 标题字体大小20sp，与顶部距离20dp，居中；
- 图片与上下控件的间距均为20dp，居中；
- 输入框整体距左右屏幕各间距20dp，内容（包括提示内容）如图所示，内容字体大小18sp；
- 按钮与输入框间距10dp，文字大小18sp。按钮背景框左右边框与文字间距10dp，上下边框与文字间距5dp，圆角半径180dp，背景色为**#3F51B5**；
- 四个单选按钮整体居中，与输入框间距10dp，字体大小18sp，各个单选按钮之间间距10dp，默认选中的按钮为第一个。

---

## 三、课堂实验结果
### (1)实验截图
  
![](/img/in-post/post-Android/lab1/截图/lab1_week4_1.png)
![](/img/in-post/post-Android/lab1/截图/lab1_week4_3.png)

### (2)实验步骤以及关键代码

- 实验步骤

    进行UI设计一定要做好相应的约束才能布局，本次实验UI的控件共四部分：
    - 标题的text，我们使用TextView组件来实现，并通过
    `app:layout_constraintLeft_toLeftOf="parent"`
    `app:layout_constraintRight_toRightOf="parent"`
        来实现标题居中。
    - 中山大学图标，通过ImageView组件来添加图片，图片我放在/res/drawable目录下，居中方式同上。
    - 搜索框，由两部分组成————输入栏和搜索按钮
        - 输入框，EditText，提示内容由`hits`修改
        - 搜索按钮，Button，圆角按钮的实现通过在drawable目录下的`shape.xml`文件来实现。
        - 将输入框和搜索按钮通过`layout_constraintTop_toTopOf` 实现并行。
        - 注意点就是对输入框的长度，我们选择`0dp`，原因我们在后面会解释。
    - 四个单选按钮，实现使用的是RadioGroup组件，并于其中添加四个RadioButton按钮，我们只需在RadioGroup的属性中添加`android:orientation="horizontal"`即可实现四个按钮平行分布，然后通过`app:layout_constraintTop_toBottomOf="@id/editText"`约束在搜索框之下，然后在调整margin至满足题目的要求。

- 关键代码：

    - 搜索栏：
        ```xml
        <EditText
            android:id="@+id/editText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            app:layout_constraintTop_toBottomOf="@id/imageView"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toLeftOf="@id/search_button"
            android:layout_marginTop="20dp"
            android:layout_marginRight="10dp"
            android:textSize="18sp"
            android:layout_marginLeft="20dp"
            android:textAlignment="center"
            android:inputType="text"
            android:hint="请输入搜索内容"/>

        <Button
            android:id="@+id/search_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginRight="20dp"
            android:paddingLeft="10dp"
            android:paddingTop="5dp"
            android:paddingRight="10dp"
            android:paddingBottom="5dp"
            android:text="搜索"
            android:textSize="18sp"
            app:layout_constraintRight_toRightOf="parent"
            android:background="@drawable/shape"
            android:textColor="@android:color/white"
            app:layout_constraintTop_toTopOf="@id/editText" />
        ```

    - RadioGroup:
        ```xml
        <RadioGroup
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:gravity="center_vertical"
            android:orientation="horizontal"
            ...
            ...
            app:layout_constraintTop_toBottomOf="@id/editText"
            >
        ```

    - shape.xml:（按钮的圆角和颜色都于此设定）
        ```xml
            <?xml version="1.0" encoding="utf-8"?>
            <shape xmlns:android="http://schemas.android.com/apk/res/android"
                android:shape="rectangle" >

                <corners android:radius="180dp" />
                <solid android:color="#3F51B5"/>

            </shape>
        ```


### (3)实验遇到的困难以及解决思路

这次UI的实验遇到的主要问题有三个：
1. 输入框的`0dp`问题
    - 对于实验所要求布局，我在思考输入框的长度该如何去匹配，先满足与左边边界还有右边按钮的margin的要求，接下来剩下的空间应该都分配给输入框的长度，可是对于`wrap_content`，这属性是只分配输入框中字的长度，即“按需分配”；对于`full_parent`，是将父组件填满，也不满足需求；所以我们使用`0dp`来使输入框的下划线填充未分配的剩余空间，即可满足题目要求。
2. `android:inputType="text"`
    - 为什么我们在输入框中添加了这么一个属性呢？并不是多余的，因为在不限制输入属性的情况下，我们输入回车的时候输入框也会读取进去，从而导致输入框向下移动，我们的布局也因此被破坏，具体看下图：
    ![](/img/in-post/post-Android/lab1/截图/lab1_week4_2.png)
3. 圆角按钮：
    - android的圆角边框是通过背景图片来设置的，圆角按钮的实现需额外添加xml文件来设置，这部分也是一开始比较困难的部分，因为没有在Button中找到对应的属性，后面搜索了一下才知道了此解决方法。

---

## 四、课后实验结果

  ![](/img/in-post/post-Android/lab1/截图/lab1_week4_1.png)

  上图为我本次实验的实验结果，与实验要求基本吻合，UI界面设计基本完成。

---

## 五、实验思考及感想

实验中的思考与感悟：

- 安卓起步阶段的配置环境确实是让我们花了不少心思，但是开始确确实实接触具体项目的时候发现AS还是非常便于上手的，安卓的预览编程真的非常舒服，还有安卓对约束`constraint`的应用使我们对控件的布局变得非常简单，而且美观，xml还有java文件使我们的项目结构更加的清晰，而且还能连上安卓机直接使用我们的半成品，提前做完后确实是有挺大的满足感的。   

- 这是我们学习安卓的第一个项目的第一个部分，虽然只是UI界面的设计，但是对于初学Android的我来说确实是一门很好的作业，这次作业之后我对AndroidStudio的设计和操作，还有AS中的基础控件的使用都基本了解清楚了，为后续更难的其他安卓编程打好了坚实的基础，希望后面的每一周任务都能如此一样提前完成，继续加油。

---

