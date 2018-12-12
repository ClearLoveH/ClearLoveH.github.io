---
layot:     post
title:      "数据存储应用开发"
subtitle:   "数据存储（二）"
date:       2018-11-12 13:10
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

### 数据存储（二）

---

## 二、实现内容

- 实验要求

    - 技术要求：
        使用SQLite数据库保存用户的相关信息和评论的相关信息，使得每次运行程序都可以使用数据库进行用户的登陆与注册，以及显示数据库中的评论；
        使用ContentProvider来获取对应用户的电话号码；
    - 功能要求：
        本次实验演示应包含2个Activity。
        - 首页Activity包含登录功能和注册功能，通过radioButton在两个页面进行切换,在登陆界面输入正确的用户名和密码后跳转到评论页面。
        - 评论Activity,界面由ListView、EditText和Button组成，ListView中展示数据库中保存的评论信息，在EditText写评论，点击Send按钮发送评论。
        - 首页Activity：
            - 应用启动时，界面初始化为登录界面，通过Login和Register两个RadioButton进行登录与注册之间的切换。
            - 点击Login切换到登录界面，可以保留注册界面时的Username，但不保存密码：
            - OK按钮点击后：
            - 若Username为空，则发出Toast提示。
            - 若Password为空，则发出Toast提示。
            - 若Username不存在，则发出Toast提示。
            - 若密码不正确，则发出Toast提示。
            - CLEAR按钮点击后：清除两个输入框的内容。
            - 点击Register切换到注册页面，可以保留登录界面时的Username，但不保存密码，在输入框和RadioButto之间存在一个头像ImageView，水平居中：
            - OK按钮点击后：
            - 若Username为空，则发出Toast提示。
            - 若New Password为空，则发出Toast提示。
            - 若New Password与Confirm Password不匹配，则发出Toast提示。
            - 若Username已经存在，则发出Toast提示。
            - CLEAR按钮点击后：清除三个输入框的内容。
        - 评论页面：
            - 界面底部有一个EditText和一个按钮，高度一致，EditText占据按钮左边的全部空间。上方的全部剩余空间由一个ListView占据（保留margin）。
            - ListView中的每条Item，包含了头像、点赞按钮这两个ImageView和用户名、评论时间、评论内容、点赞数这4个TextView。
            - 用户名、评论时间、评论内容在头像的右边。
            - 点赞按钮在Item的最右边，而且在用户名+评论时间的总高度上处于竖直方向上居中，注意：总高度不包括评论占据的高度
            - 点赞数在点赞按钮的左边，竖直方向居中要求同点赞按钮。
            - 点击EditText写评论
            - 点击Send按钮发送评论
            - 若EditText为空，则发出Toast提示。
            - 若EditText不为空，则发送评论，在数据库和ListView中添加新评论。
            - ListView中的Item点击事件：
                - 短按评论：弹出对话框，显示该评论的用户以及通讯录中该用户的电话号码。图3.4.
                - 长按评论：
                    - 若该评论为当前用户发送的，弹出是否删除的对话框,若选择了Yes，则删除该条评论并更新数据库和ListView。
                    - 若该评论不为当前用户发送的，弹出是否举报的对话框，若选择了Yes，则弹出Toast提示，不需做任何数据库和ListView的更改。

- 实验目的
    - 学习SQLite数据库的使用。
    - 学习ContentProvider的使用。
    - 复习Android界面编程。

---

## 三、课堂实验结果
### (1)实验截图
    

- 点击Login切换到登录界面
    - 若Username为空，则发出Toast提示
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(1).png)
    - 若Password为空，则发出Toast提示	
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(2).png)
    - 若Username不存在，则发出Toast提示
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(3).png)
    - 若密码不正确，则发出Toast提示
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(8).png)	

- 点击Register切换到注册页面	
    - 若Username为空，则发出Toast提示
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(4).png)
    - 若New Password为空，则发出Toast提示	
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(5).png)
    - 若New Password与Confirm Password不匹配，则发出Toast提示
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(6).png)
    - 注册成功
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(7).png)
    - 若Username已经存在，则发出Toast提示	
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(9).png)

- 评论页面	
    - 若EditText为空，则发出Toast提示
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(11).png)
    - 短按评论：弹出对话框，显示该评论的用户以及通讯录中该用户的电话号码	
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(12).png)
        - 我们再通讯录中没有该用户号码，所以显示'not exist'
        - 在手机通讯录中添加一个名为Q的用户号码，再短按评论。
        ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(27).png)
        ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(28).png)
    - 长按评论：
        - 若为当前用户的评论，长按弹出是否删除的对话框
        ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(13).png)
        ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(14).png)
        - 若不为当前用户的评论，长按弹出是否举报的对话框
        ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(19).png)

- `（加分项实现效果展示）`
    - 头像——在用户注册页面可以选择用户头像，ImageView初始化为图，点击ImageView，进入手机图库进行图片选择。
    - 新建一个带头像的用户：（不选择头像的注册用户提供有默认头像）
        - 进入手机图库进行图片选择	
        ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(15).png)
        - ImageView显示本次选择的图片
            - 如果正确选择了一张图片，则ImageView显示本次选择的图片。
            - 如果没有正确选择图片（如在图片选择页面点击了取消或按了手机的BACK键），则ImageView保留本次点击事件发生前的状态，如初始的＋号图片，或者是上一个被正确选择的图像。
        ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(16).png)


    - 在评论页面，每条Item应当正确显示用户的头像，如果用户没有在注册页面选择头像，则使用默认头像。
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(18).png)

    - 点赞——在评论界面，点赞按钮可以被点击，点赞数可以正常统计，用户点赞状态可以被正常记录，白色的未点赞状态，经用户点击后变为红色的点赞状态，点赞数加1；再次点击点赞按钮可取消点赞，点赞数减1。要求用数据库记录点赞的信息，使得应用重启后用户的点赞状态，评论的点赞数可以正常显示，注意：用户的对每条评论的点赞状态是唯一的，即不同用户对每条评论的点赞状态应当分开记录，同一用户对不同评论的点赞状态也应当分开记录。同理，每条评论的点赞数也应当分开记录。
    - 对所有评论进行点赞：
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(21).png)
    - 登陆Q用户，查看评论页，发现点赞数保存，但由于都是非Q的点赞，所以点赞按钮并没有显示红色
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(22).png)
    - 新建W用户：
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(23).png)
    - 发表评论并对自己的评论点赞
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(24).png)
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(25).png)
    - 再重新登陆1用户，发现已经点赞过的前两条评论，点赞按钮状态依旧保持，不是自己点赞的就为灰色状态，一切正常
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(26).png)
    
 
    
    
    
### (2)实验步骤以及关键代码
- 主页Activity——主页Activity主要实现注册及登陆事件。通过boolean变量`loginOrRegister`来判断当前是登陆状态还是注册状态（这变量的切换在RadioGroup的`setOnCheckedChangeListener`中完成，同时切换时改变主页的UI界面）（数据库实现放在加分项中阐述）
    - RadioGroup的`setOnCheckedChangeListener:
        ```java
        mode.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
                @Override
                public void onCheckedChanged(RadioGroup group, int checkedId) {
                    if(checkedId==Login.getId()){
                        //调整UI
                        add.setVisibility(View.GONE);
                        confirmPassword.setVisibility(View.GONE);
                        password.setHint("Password");
                        loginOrRegister=true;
                    }
                    else{
                        //调整UI
                        add.setVisibility(View.VISIBLE);
                        confirmPassword.setVisibility(View.VISIBLE);
                        password.setHint("New Password");
                        loginOrRegister=false;
                    }
                }
            });
        ```
    - 登陆： 
        ```java
        //判断注册还是登陆
        if (loginOrRegister){
            if(login(userN,newP)){
                intent=new Intent(MainActivity.this,CommentActivity.class);
                Bundle bundle=new Bundle();
                bundle.putString("loginUser",userN);
                bundle.putString("icon","");
                intent.putExtras(bundle);
                startActivity(intent);
            }
        }              
        ```
        ```java
        //登陆
        public boolean login(String username,String password){
            String sql="select * from user where username=? ";
            String sql2="select * from user where username=? and password=?";
            Cursor cursorUsername=mDatabase.rawQuery(sql, new String[]{username});
            Cursor cursorPassword=mDatabase.rawQuery(sql2, new String[]{username,password});
            if(cursorUsername.moveToFirst()==false){
                ······
            }
            else {
                ······
            }
        }
        ```
        
    - 注册：    
        ```java
        else {
            if(!newP.equals(confirmPassword.getText().toString())){
                toast=Toast.makeText(getApplicationContext(),"Password Mismatch.",Toast.LENGTH_SHORT);
                toast.show();
            }
            else if(register(userN, newP)){
                //注册成功toast提示
                toast=Toast.makeText(getApplicationContext(),"Register Successfully.",Toast.LENGTH_SHORT);
                toast.show();
                mode.check(R.id.Login);
            }
        }
        ```java
        //注册
        public boolean register(String username,String password){
            //判断该用户是否已经注册
            String sql1="select * from user where username=? ";
            Cursor cursorUsername=mDatabase.rawQuery(sql1, new String[]{username});
            if(cursorUsername.moveToFirst()==true){
                toast=Toast.makeText(getApplicationContext(),"Username already existed.",Toast.LENGTH_SHORT);
                toast.show();
                return false;
            }
            String path=imagePath;
            String sql2="insert into user(username,password,iconPath) values(?,?,?)";
            Object obj[]={username,password,path};
            mDatabase.execSQL(sql2, obj);
            return true;
        }
        ```


- 评论Activity——主要实现评论的呈现、点赞事件。
    - 该Activity主要任务是在listView的item的点击事件（长按短按），以及点赞按钮的点击事件，和评论事件的实现。——这里有一个注意点是我们需要把item中的点赞按钮在xml文件中设定一个属性`focusable="false" `目的是不让点赞的按钮获取到整个item的点击的焦点，否则会导致item长按短按失效。
        ```java
        //listView点击事件——点击与长按
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                String number = "";
                if (ContextCompat.checkSelfPermission(CommentActivity.this, Manifest.permission.READ_CONTACTS)
                        != PackageManager.PERMISSION_GRANTED) {
                    //获取访问通讯录权限             
                    if (ActivityCompat.shouldShowRequestPermissionRationale(CommentActivity.this,
                            Manifest.permission.READ_CONTACTS)) {
                        ······
                    }
                } else {
                    //读取通讯录
                    Cursor cursor = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null, ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME + " = \"" + data.get(position).getName() + "\"", null, null);
                    if(cursor.moveToFirst()){
                        do {
                            number += cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER)) + "\n";
                        } while (cursor.moveToNext());
                    }
                    //输出通讯录信息
                    ······
                    ······
                }
            }
        });
        listView.setOnItemLongClickListener(new AdapterView.OnItemLongClickListener() {
            @Override
            public boolean onItemLongClick(AdapterView<?> parent, View view, int position, long id) {
                //如果该长按的评论是属于当前用户发表的，弹出是否删除对话框
                ······
                    builder.setPositiveButton("Yes", new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            //删除该评论
                            //通过 _id 而非是id来获取长按的item的id值
                            //删除之后对后面的行数进行ID更新，update -1
                            //其实可以用评论时间来当删除操作的键
                            mDatabase.execSQL("DELETE FROM comment WHERE _id = "+ temp );
                            mDatabase.execSQL("UPDATE comment SET _id = _id -1 WHERE _id > " + temp);
                            refresh();
                        }
                    });
                    builder.show();
                    return true;
                }
                //该评论不是当前用户发表的，长按弹出举报。
                else{
                    ······
                }
            }
        });
        ```
    - `refresh()`是我自己写的用于更新adapter的函数。
        ```java
        //刷新当前activity
        //先把data中清空，再重新加载
        public void refresh(){
            data.clear();
            initData();
            myAdapter.notifyDataSetChanged();
        }
        ```
    - 发送评论——send按钮事件：
        ```java
        Send.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    String content=editText.getText().toString();
                    if(content.equals("")){
                        ······
                    }
                    else{
                        Toast.makeText(getApplicationContext(),"Comment Successfully.",Toast.LENGTH_SHORT);
                        Date date = new Date();
                        DateFormat df=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                        String time="At: "+df.format(date);
                        String sql="insert into comment(username,time,content,likedBy) values(?,?,?,?)";
                        Object obj[]={currentUser,time,content,""};
                        mDatabase.execSQL(sql, obj);
                        //设置头像
                        String sql1="select * from user where username=? ";
                        Cursor cursorUsername=mDatabase.rawQuery(sql1, new String[]{currentUser});
                        if(cursorUsername.moveToFirst()==true){
                            String path=cursorUsername.getString(cursorUsername.getColumnIndex("iconPath"));
                            if(null!=path) {
                                Uri uri = Uri.parse(path);
                            }
                        }
                        refresh();
                        editText.setText("");
                    }
                }
            });
        ```


- 本次实验的`加分项`：
    （在实验截图中已经呈现所实现效果）
    - 加分项中我对于评论头像的存储以及点赞数状态的保存都是通过`数据库存储`的。下面是我的数据库：主要由两个表组成，一个`user`表存储注册好的用户信息，user中的iconPath列存储的是用户头像；另一个`comment`表，用于存储评论信息，likedBy列存储的是该评论点赞的所有用户名，其中不同的用户通过`,`隔开，取用时通过String类型的`split(",")`函数来获得用户名数组，点赞数即为点赞用户的个数。
        ```java
        public class myDB extends SQLiteOpenHelper {
            private static final String DB_NAME= "myDB";
            private static final String TABLE_NAME = "user";
            private static final int DB_VERSION = 1;

            public myDB(Context context){
                super(context, DB_NAME, null, DB_VERSION);
            }
            @Override
            public void onCreate(SQLiteDatabase sqLiteDatabase) {
                String CREATE_TABLE = "CREATE TABLE if not exists "
                        + TABLE_NAME
                        + " (_id INTEGER PRIMARY KEY, username TEXT, password TEXT, iconPath TEXT)";
                sqLiteDatabase.execSQL(CREATE_TABLE);
                String CREATE_TABLE_Comment = "CREATE TABLE if not exists comment (_id INTEGER PRIMARY KEY, username TEXT, time TEXT, content TEXT, likedBy TEXT)";
                sqLiteDatabase.execSQL(CREATE_TABLE_Comment);
            }

            //onUpgrade在本次实验不需要用到，但Android要求重写才能实例化
            @Override
            public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int ii) {}

        }
        ```
    - 对于点赞状态，即评论是否被当前登陆用户所点赞，只需在获得的点赞用户数组判断是否包含当前登陆用户即可，然后把当前用户是否点赞某个评论的结果保存为一个Boolean值变量`ifLiked`存储在comment类中，当前登陆用户我通过一个静态String变量`currentUser`来存储，点赞需要单独实现一个。

        `我的comment类：`
        ```java
        public class CommentInfo implements Serializable {
            private String name;
            private String time;
            private String content;
            private String likedBy;
            private String iconPath;
            private Boolean ifLiked;
            CommentInfo(String name,String time,String content,String likedBy,Boolean ifLiked,String iconPath){
                this.name=name;
                this.time=time;
                this.content=content;
                this.likedBy=likedBy;
                this.ifLiked=ifLiked;
                this.iconPath=iconPath;
            }
            ···
            ···
        }
        ```

        `myAdapter.java:`
        ```java
        public abstract class myAdapter extends BaseAdapter {
            private Context mContext;
            private List<CommentInfo> mList = new ArrayList<>();
            //构造函数
            ···
            ···
            @Override
            //getView中对item的控件进行绑定
            public View getView(final int i, View view, ViewGroup viewGroup) {
               ···
               ···
                //头像设置
                if(mList.get(i).getIconPath()!=null && !mList.get(i).getIconPath().equals("")){
                    Bitmap bmp = BitmapFactory.decodeFile(mList.get(i).getIconPath());
                    viewHolder.icon.setImageBitmap(bmp);
                }
                ···
                //通过一个Boolean值判断当前用户是否点赞
                if(mList.get(i).getIfLiked())
                    viewHolder.likeButton.setBackgroundResource(R.drawable.red);
                else viewHolder.likeButton.setBackgroundResource(R.drawable.white);
                //储存规则——每添加一个点赞者，likedBy+= username +",";
                //所以通过逗号数来判断点赞数。
                if(mList.get(i).getLikedBy().equals(""))
                    viewHolder.likeNumber.setText("0");
                else {
                    String []split=mList.get(i).getLikedBy().split(",");
                    String count= String.valueOf(split.length);
                    viewHolder.likeNumber.setText(count);
                }

                ······
            }
            /**
            * 点赞的事件监听器
            */
            ···
            ···
            class ViewHolder {
                TextView commentName;
                TextView commentTime;
                TextView commentContent;
                TextView likeNumber;
                ImageView icon;
                Boolean ifLikedByCurUser;
                Button likeButton;
            }
           ···
           ···
        }
        ```

    

### (3)实验遇到的困难以及解决思路

本周的遇到的问题我都有记录下来，可以说是问题比较多的了：

- `_id 获取点击的item的id值`
    - 我们通过长按与点击commentList中的item，触发对应的事件时，有一个问题就是：listView中adapter的getView函数中的`final int i`是用于获取当前item在list中的id的，范围是从0~list的size()-1。
    - 而我们在通过数据库来输出所有评论的时候，是通过其键值`_id`(注意不能使用`id`)，来访问的，但是在查看应用启动后产生的数据库中发现，`_id`的范围是从1~list.size()，所以我们使用_id的时候注意与listview中i的关系，否则会导致点击某个评论触发的是上一个评论的事件。

- 而在删除评论之后，因为我们使用的是键值ID，需要对ID进行更新，才不会影响后续的删除操作（如果有和其他表有关联不能随便改主键id，单表就无所谓了）
    - 注意_id 与 i的关系（_id : 0->i-1）

- 对通讯录权限的处理
    - 对通讯录权限的处理相对比较麻烦，光在manifest文件中申请权限是不够的，这里我的处理方式借用了CSDN[此博客](https://blog.csdn.net/njweiyukun/article/details/50044193)上的方式。



- `setText（）`不能直接用int——用String.valueof函数


- 不能使用string自带的contains函数来判断我存取的`likeBy`字符串中是否包含当前用户（尽管我已经把每个用户通过","分离），这个是判断有没有指定子串，若某用户的Uesrname包含了另外一个username，这种方法会导致判断出错，还是需要把likedBy先分解再判断。


    
- 如果imageView中是src属，setImageBitmap(bitmap);会覆盖了Src属性---就是说只有一张图；
而如果imageView中是background属，setImageBitmap(bitmap);会重复background的属性。--会有两张图
    ```xml
    <ImageView
        android:id="@+id/iv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:src="@drawable/ic_launcher_background"
        />
    ```

- 注意权限申请问题，要在手机内设置，这个坑是非常坑人的！！！虽然我们在manifest文件中已经申请了读写以及访问通讯录之类的权限，但是我们还要再手机中给权限，如下，否则我们还是没办法访问手机内存中的图。
    ![](/img/in-post/post-Android/lab3/截图/lab3_part2/lab3_part2_1_(30).png)

---

## 四、实验思考及感想

- 这周的数据存储（二）与上周的sharedPreferences和File两个数据存储方式完全不是一个等级的，二者之间的代码量与代码难度相差太大，本周任务是比较重的，但是好在我没有拖延，提前一周完成。项目过程中遇到不少的问题烦恼着我，都记录在上面了，这次实验遇到了多少难题，就可以说是学习到了多少新的知识，虽然第一次在安卓中使用数据库熟练度不足，但是在使用过后就会发现数据库对应用的数据保存以及提取可谓是有条不紊，感想主要是任务比较重，代码量挺多，但是因此学习到的知识也多，虽然提前完成了本周任务，但是我不能膨胀，要继续虚心耐心地学习下去。
- 安卓中数据存储的四大主要方式：
    1. SharedPreferences
    2. SQLite
    3. Content Provider
    4. File

    本周学习了其中的2，3，在安卓上是实用性比较高的，为我在后面期中项目的完成提供了不少的帮助，希望在后续学习到的其他项目可以继续丰富我的知识。
---


