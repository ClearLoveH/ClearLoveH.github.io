---
layot:     post
title:      "数据存储应用开发"
subtitle:   "数据存储（一）"
date:       2018-11-7 11:32
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Android
---

>Invictus Gaming——We are the champion！.

---

[项目仓库](https://gitee.com/ClearLoveH/PersonalProject3)

---
## 一、实验题目

### 数据存储（一）

---

## 二、实现内容

- 实验要求
    - 本次实验演示应用包含两个Activity。
    - 首先是密码输入Activity：
        - 若应用首次启动，则界面呈现出两个输入框，分别为新密码输入框和确认密码输入框。
        - 输入框下方有两个按钮：
            - OK按钮点击后：
                - 若New Password为空，则发出Toast提示。
                - 若New Password与Confirm Password不匹配，则发出Toast提示。
                - 若两密码匹配，则保存此密码，并进入文件编辑Activity。
            - CLEAR按钮点击后：清楚两输入框的内容。
        - 完成创建密码后，退出应用再进入应用，则只呈现一个密码输入框。
            - 点击OK按钮后，若输入的密码与之前的密码不匹配，则弹出Toast提示。
            - 点击CLEAR按钮后，清除密码输入框的内容。
        - 出于演示和学习的目的，本次实验我们使用SharedPreferences来保存密码。但实际应用中不会使用这种方式来存储敏感信息，而是采用更安全的机制。见[这里](https://stackoverflow.com/questions/1925486/android-storing-username-and-password)和[这里](https://stackoverflow.com/questions/785973/what-is-the-most-appropriate-way-to-store-user-settings-in-android-application/786588)。
    - 文件编辑Activity：
        - 界面底部有三个按钮，高度一致，顶对齐，按钮水平均匀分布，三个按钮上方除ActionBar和StatusBar之外的全部空间由一个EditText占据（保留margin）。EditText内的文字竖直方向置顶，左对齐。
        - 在编辑区域输入任意内容，点击SAVE按钮后能保存到指定文件（文件名随意）。成功保存后，弹出Toast提示，。
        - 点击CLEAR按钮，能清空编辑区域的内容。
        - 点击LOAD按钮，能够从同一文件导入内容，并显示到编辑框中。若成功导入，则弹出Toast提示。若读取文件过程中出现异常（如文件不存在），则弹出Toast提示。
    - 特殊要求：进入文件编辑Activity后，若点击返回按钮，则直接返回Home界面，不再返回密码输入Activity。

- 实验目的
    - 学习SharedPreference的基本使用。
    - 学习Android中常见的文件操作方法。
    - 复习Android界面编程。

---

## 三、课堂实验结果
### (1)实验截图
- Figure 1：首次进入，呈现创建密码界面。
    ![](/img/in-post/post-Android/lab3/截图/lab3_week9_(8).png)
- Figure 2：若密码不匹配，弹出Toast提示。
    ![](/img/in-post/post-Android/lab3/截图/lab3_week9_(10).png)
- Figure 3：若密码为空，弹出Toast提示。
    ![](/img/in-post/post-Android/lab3/截图/lab3_week9_(9).png)
- Figure 4：退出后第二次进入呈现输入密码界面。
    ![](/img/in-post/post-Android/lab3/截图/lab3_week9_(2).png)
- Figure 5：若密码不正确，弹出Toast提示。
    ![](/img/in-post/post-Android/lab3/截图/lab3_week9_(3).png)
- Figure 6：文件加载失败，弹出Toast提示。
    ![](/img/in-post/post-Android/lab3/截图/lab3_week9_(5).png)
- Figure 7：成功保存文件，弹出Toast提示。
    ![](/img/in-post/post-Android/lab3/截图/lab3_week9_(6).png)
- Figure 8：成功导入文件，弹出Toast提示。
    ![](/img/in-post/post-Android/lab3/截图/lab3_week9_(7).png)
### (2)实验步骤以及关键代码
- 对密码输入的Activity:
    - 未注册时候的OK Button的点击事件——密码为空，密码不匹配，密码配对三种情况：
        ```java
        OK.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        String newP=newPassword.getText().toString();
                        String confirmP=confirmPassword.getText().toString();
                        if(newP.length()==0){
                            toast=Toast.makeText(getApplicationContext(),"Password cannot be empty.",Toast.LENGTH_SHORT);
                            toast.show();
                        }
                        else if(!newP.equals(confirmP)){
                            toast=Toast.makeText(getApplicationContext(),"Password Mismatch.",Toast.LENGTH_SHORT);
                            toast.show();
                        }
                        else{
                        //此部分我把注册好的password和一个表明是否注册的Boolean变量通过SharedPreferences存储起来
                            SharedPreferences.Editor editor = sharedPreferences.edit();
                            editor.putString("Password", newP);
                            editor.putBoolean("ifRegister",true);
                            editor.commit();
                            intent=new Intent(MainActivity.this,FileEditActivity.class);
                            startActivity(intent);
                        }
                    }
                });
        ```
    - 注册后重新进入app中，首页只剩下输入密码的界面，同时OK Button的点击事件也会进行更改，通过上一步传入的Boolean变量`ifRegister`来判断是否注册过，缺省值为false，如果未注册（ifRegister == false）则为第一步的实现。
        ```java
        Boolean ifRegister=sharedPreferences.getBoolean("ifRegister",false);
            if(ifRegister) {
                newPassword.setVisibility(View.GONE);
                confirmPassword.setHint("Password");
                OK.setOnClickListener(new View.OnClickListener(){
                    @Override
                    public void onClick(View v) {
                        String password=sharedPreferences.getString("Password","");
                        if(password.equals(confirmPassword.getText().toString())){
                            intent=new Intent(MainActivity.this,FileEditActivity.class);
                            startActivity(intent);
                        }
                        else{
                            toast = Toast.makeText(getApplicationContext(), "Invalid Password.", Toast.LENGTH_SHORT);
                            toast.show();
                        }
                    }
                });
            }
            else{
                ···
                ···
                ···
            }
        ```
- 对文件编辑的Activity：
    - Save Button，把EditText中当前输入的内容通过`FileOutputStream`存取至自己命名的文件中
        ```java
        Save.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                try (FileOutputStream fileOutputStream = openFileOutput("data.txt", MODE_PRIVATE)) {
                    String str = FileText.getText().toString();
                    fileOutputStream.write(str.getBytes());
                    toast=Toast.makeText(getApplicationContext(),"Save successfully.",Toast.LENGTH_SHORT);
                    toast.show();
                    Log.i("TAG", "Save successfully.");
                } catch (IOException ex) {
                    Log.e("TAG", "Fail to save file.");
                }
            }
        });
        ```
    - Load Button，通过`FileInputStream`读取文件，获取其中的内容，若获取不到文件（Fail to load）也需要弹出提示的Toast。
        ```java
        Load.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                try (FileInputStream fileInputStream = openFileInput("data.txt")) {
                    byte[] contents = new byte[fileInputStream.available()];
                    fileInputStream.read(contents);
                    //gradle 添加依赖    implementation 'org.apache.httpcomponents:httpcore:4.4.4'
                    String fileContent = EncodingUtils.getString(contents, "UTF-8");
                    FileText.setText(fileContent);
                    toast=Toast.makeText(getApplicationContext(),"Load successfully.",Toast.LENGTH_SHORT);
                    toast.show();
                } catch (IOException ex) {
                    toast=Toast.makeText(getApplicationContext(),"Fail to read file.",Toast.LENGTH_SHORT);
                    toast.show();
                    Log.e("TAG", "Fail to read file.");
                }
            }
        });
        ```
- 本次实验还有一个要求便是，进入到文件编辑的Activity时，我们不需要返回登录的Activity，返回直接退出app，这里需要实现这个效果，我们只需要在`AndroidManifest.xml`文件的MainActivity中添加一个属性`android:noHistory="true"`即可，这个属性使MainActivity在离开时就会自动销毁，后续不会再返回到登录页面了。
    ```xml
    <activity
            android:name=".MainActivity"
            android:noHistory="true">  
    ```
### (3)实验遇到的困难以及解决思路
- 在使用`FileInputStream`来读取文件内容时，有一个函数是获取文件内容所必须的——`EncodingUtils.getString(contents, "UTF-8");`，但是使用却会报错，这时候我们需要在gradle中添加依赖——` implementation 'org.apache.httpcomponents:httpcore:4.4.4'`，在重新sysc即可。

- 安卓版本的问题，新建了一个项目的minSDK设置的太高了导致已经下载的虚拟机版本跟不上，只需在build.gradle中修改`minSdkVersion`即可。

---

## 四、实验思考及感想

- 这周任务是我们第一周的安卓的数据存储任务，主页目的是让我们去体验一下简单的安卓数据存储，但实用性有限，毕竟用`sharedPreferences`来存储密码等用户信息还是不太安全的，所以本周的任务相对轻松。
- 这周的sharedPreferences和File两个数据存储方式，主要是给下周的数据存储（二）做铺垫，下周的任务量完全与此周不是一个等级的，但是本周的也让我对安卓的数据存储有了初步的了解，下周任务量比较大，不膨胀，继续学习下去。
- 安卓中数据存储的四大主要方式：
    1. SharedPreferences
    2. SQLite
    3. Content Provider
    4. File

    本周学习了其中的1，4，下周对SQLite的使用会更加复杂，数据储存在安卓实战中的应用是很多的，我们此次的期中项目也会涉及，所以还需继续后续的学习。

---

