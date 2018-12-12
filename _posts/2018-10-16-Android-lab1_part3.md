---
layot:     post
title:      "中山大学智慧健康服务平台应用开发"
subtitle:   "Intent、Bundle的使用以及RecyclerView、ListView的应用"
date:       2018-10-16 12:55
author:     "Heng"
header-img: "img/弗雷尔卓德2.jpg"
catalog: true
tags:
    - Android
---

>Royal may give up now..

---

[项目仓库](https://gitee.com/ClearLoveH/PersonalProject1)

---

## 一、实验题目 

### Intent、Bundle的使用以及RecyclerView、ListView的应用

实验目的：
- 复习事件处理。
- 学习Intent、Bundle在Activity跳转中的应用。
- 学习RecyclerView、ListView以及各类适配器的用法。
- 学习FloatingActionBar的用法。

---

## 二、实现内容

本次实验模拟实现一个健康食品列表，有两个界面，第一个界面用于呈现食品列表 如下所示

![](/img/in-post/post-Android/lab1/截图/lab1_week6_要求1.jpg)

- 点击右下方的悬浮按钮可以切换到收藏夹
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_要求2.jpg)


- 上面两个列表点击任意一项后，可以看到详细的信息：
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_要求3.jpg)

UI要求：
- 食品列表
    - 每一项为一个圆圈和一个名字，圆圈和名字都是垂直居中。圆圈内的内容是该食品的种类，内容要处于圆圈的中心，颜色为白色。食品名字为黑色，圆圈颜色自定义，只需能看见圆圈内的内容即可。
- 收藏夹
    - 与食品列表相似
- 食品详情界面
    - 界面顶部
    顶部占整个界面的1/3。每个食品详情的顶部颜色在数据中已给出。返回图标处于这块区域的左上角，食品名字处于左下角，星标处于右下角，边距可以自己设置。 返回图标与名字左对齐，名字与星标底边对齐。 建议用RelativeLayout实现，以熟悉RelativeLayout的使用。
    - 界面中部
    使用的黑色argb编码值为#D5000000，稍微偏灰色的“富含”“蛋白质”的argb编码值为#8A000000。"更多资料"一栏上方有一条分割线，argb编码值为#1E000000。右边收藏符号的左边也有一条分割线，要求与收藏符号高度一致，垂直居中。字体大小自定。"更多资料"下方分割线高度自定。这部分所有的分割线argb编码值都是#1E000000。
    - 界面底部
    使用的黑色argb编码值为#D5000000。
- 标题栏
    - 两个界面的标题栏都需要去掉


功能要求：
- 使用RecyclerView实现食品列表。点击某个食品会跳转到该食品的详情界面，呈现该食品的详细信息。长按列表中某个食品会删除该食品，并弹出Toast，提示 "删除XX" 。
- 点击右下方的FloatingActionButton，从食品列表切换到收藏夹或从收藏夹切换到食品列表，并且该按钮的图片作出相应改变。
- 使用ListView实现收藏夹。点击收藏夹的某个食品会跳转到食品详情界面，呈现该食品的详细信息。长按收藏夹中的某个食品会弹出对话框询问是否移出该食品，点击确定则移除该食品，点击取消则对话框消失。如长按“鸡蛋”。
- 商品详情界面中点击返回图标会返回上一层。点击星标会切换状态，如果原本是空心星星，则会变成实心星星；原本是实心星星，则会变成空心星星。点击收藏图表则将该食品添加到收藏夹并弹出Toast提示 "已收藏" 。

---

## 三、课堂实验结果

### (1)实验截图

- 初始界面UI：
![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现1.png)

- 进入收藏夹：
![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现2.png)

- 点击列表任意一项进入详情页：
![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现3.png)

    - 点击详情页面中的星星：
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现4.png)

    - 点击详情页中的购物车收藏物品并弹出toast:
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现5.png)

    - 点击返回：
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现6.png)

- 点击收藏夹进入收藏夹页面：
![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现7.png)

    - 从收藏夹中点击该物品，依旧可以正常访问详情界面，`并且星星的状态也保存下来`——这是本次实验的加分项之一，后面其他的加分项总结中会再提到：
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现8.png)

    - 长按删除收藏夹中物品，弹出对话框：
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现9.png)

    - 确认删除:
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现10.png)

    - 返回主页。

- 主页中长按某物品，删除该物品，并弹出相应toast：
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_实现11.png)



### (2)实验步骤以及关键代码

先附上本次实验的项目结构：
![](/img/in-post/post-Android/lab1/截图/lab1_week6_步骤1.png)

- 主界面UI的构建：
    1. `Collection`是我创建的用于封装与列表中所有item相关的变量的类。
        ```java
        //Collection类的实现
        public class Collection implements Serializable {
            private static final long  serialVersionUID=1L;
            public String circle ;
            public String name ;
            public String type;
            public String nutrition;
            public String color;
            public boolean star;
            Collection (String circle,String name,String type,String nutrition,String color, boolean star){
                this.circle=circle;
                this.name=name;
                this.type=type;
                this.nutrition=nutrition;
                this.color=color;
                this.star=star;
            }
            public String getName(){
                return name;
            }
            public String getCircle(){
                return circle;
            }
            public String getType(){return type;}
            public String getNutrition(){return nutrition;}
            public String getColor(){return color;}
            public boolean getStar(){return star;}
            //记录星星状态
            public void setStar(boolean star){this.star=star;}
        }
        ```
        - 单个item的布局（item.xml）——这里我使用的是`RelativeLayout`布局
            ```xml
            <RelativeLayout
                android:layout_width="fill_parent"
                android:layout_height="90dp">
                    <TextView
                        android:id="@+id/circle"
                        ···
                        android:text="極"
                        android:background="@drawable/circle"
                        />
                    <TextView
                        android:id="@+id/name"
                        android:layout_centerVertical="true"
                        android:layout_toRightOf="@id/circle"
                        ···
                        android:text="Royal"
                        />
            </RelativeLayout>
            ```
            样式
            ![](/img/in-post/post-Android/lab1/截图/lab1_week6_步骤2.png)
    2. 本次主页面列表实现要求使用的是`recyclerView`，关于recyclerView的知识点确实是比较多且重要的。
        - 初始化数据：
            ```java
            //初始化数据
            public void initData() {
                String[] name = new String[]{"大豆", "十字花科蔬菜", "牛奶", "海鱼", "菌菇类", "番茄", "胡萝卜", "荞麦", "鸡蛋"};
                String[] circle = new String[]{"粮", "蔬", "饮", "肉", "蔬", "蔬", "蔬", "粮", "杂"};
                String[] type=new String[]{"粮食","蔬菜","饮品","肉食","蔬菜","蔬菜","蔬菜","粮食","杂"};
                String[] nutrition =new String[]{"蛋白质","维生素C","钙","蛋白质","微量元素","番茄红素","胡萝卜素","膳食纤维","几乎所有营养物质"};
                String[] color=new String[]{"#BB4C3B","#C48D30","#4469B0","#20A17B","#BB4C3B","#4469B0","#20A17B","#BB4C3B","#C48D30"};
                for (int i = 0; i < 9; i++) {
                    Collection temp=new Collection(circle[i],name[i],type[i],nutrition[i],color[i],false);
                    data.add(temp);
                }
                Collection temp=new Collection("*","收藏夹","","","",false);
                data2.add(temp);
            }
            ```
        - 创建RecyclerView的Adapter同时实现convert类：
            ```java
            myAdapter=new myAdapter(this, R.layout.item, data) {
            //convert类具体实现
                @Override
                public void convert(myViewHolder holder, Collection s) {
                    TextView name = holder.getView(R.id.name);
                    name.setText(s.getName());
                    Button first = holder.getView(R.id.circle);
                    first.setText(s.getCircle());
                }
            };
            ```
        - 使用recyclerView来进行线性布局：
            ```java
            //foodlist
            recyclerView = findViewById(R.id.recyclerView);
            myLayoutManager=new LinearLayoutManager (this,LinearLayoutManager.VERTICAL,false);
            recyclerView.setLayoutManager(myLayoutManager);
            ```
   
    3. 收藏夹的悬浮按钮实现（这里使用的是`ConstraintLayout`布局，调整悬浮按钮的位置也使用了对应的方法）：
        ```xml
            <android.support.design.widget.FloatingActionButton
                ····
                android:id="@+id/car"
                app:layout_constraintRight_toRightOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"
                ····
                android:src="@drawable/collect"
                android:clickable="true"
                android:onClick="change"
                />
        ```
        - `change`函数于此是用于主页面与收藏夹页面之间的跳转，实现难度不大，只需将收藏夹的listView的`Visibility`属性进行调整即可——我在这里顺带添加了一个旋转效果，点击悬浮按钮切换页面改变图片的同时旋转：   
            ```java
            private boolean reverse = false;
            //flag判断当前按钮状态
            //添加旋转动画
            private boolean flag=false;
            public void change(View view) {
                car_button=view.findViewById(R.id.car);
                flag =!flag;
                if(flag) {
                    car_button.setImageResource(R.drawable.mainpage);
                    findViewById(R.id.shoucangjia).setVisibility(View.VISIBLE);
                    findViewById(R.id.recyclerView).setVisibility(View.GONE);
                }
                else ····
                //rotate
                float toDegree = reverse ? -360f : 360f;
                ObjectAnimator animator = ObjectAnimator.ofFloat(view, "rotation", 0.0f, toDegree).setDuration(400);
                animator.start();
                reverse = !reverse;
            }
            ```
     
    4. （重点）主页物品的长按删除与点击进入详情页：
        - 本次实验的最重点同时最难点就是从详情界面与主页之间的互相跳转了。这部分的页面跳转与收藏夹不同，收藏夹与主页的相互跳转是没有参数传进去的，但是详情页面不同，我们在主页点击了某个Collection之后，在详情页是需要显示对应的信息出来的，这个信息是要作为参数同时返回给详情页的，所以这部分功能的实现就用到了本次实验的核心——`Intent与bundle`。
        - 点击某个具体的Collection时，将点击的该Collection作为一个参数传递给`detailActivity`页面，然后在detailActivity页面使用该参数显示对应的文本。
            ```java
            //页面跳转所需Intent
            intent.setClass(MainActivity.this,foodListMainActivity.class);
            search_button
            ```
            ```java
            ((myAdapter) myAdapter).setOnItemClickListener(new myAdapter.OnItemClickListener() {
                @Override
                public void onClick(int position) {
                    //传参数页面转换
                    Intent intent = new Intent(foodListMainActivity.this , detailActivity.class);
                    Bundle bundle = new Bundle();
                    bundle.putInt("pos", position);
                    bundle.putSerializable("foodItem",  data.get(position));
                    intent.putExtras(bundle);
                    startActivityForResult(intent, 1);
                }

                @Override
                public void onLongClick(int position) {
                    toast= Toast.makeText(getApplicationContext(),data.get(position).getName()+"被删除", Toast.LENGTH_SHORT);
                    toast.show();
                    ((myAdapter) myAdapter).remove(position);
                }
            });
            ```
        - `detailActivity`页面怎么获取并利用参数？
            ```java
                //获取MainActivity的参数，实现有参数跳转
                temp=(Collection)getIntent().getSerializableExtra("foodItem");
                ((TextView)findViewById(R.id.foodName)).setText(temp.getName());
                ((TextView)findViewById(R.id.foodType)).setText(temp.getName());
                ((TextView)findViewById(R.id.nutrition)).setText(temp.getNutrition());

                findViewById(R.id.background).setBackgroundColor(Color.parseColor(temp.getColor()));

             ```
        - 这里涉及到本次实验的一个加分项——星星状态的保存。        我在Collection类中添加了一个boolean变量`starFlag`用于保存星星的状态，同时在点击星星时通过该变量来进行星星的变化，刚进入详情页时就通过starFlag的值来设置星星状态。
            ```java
            starFlag = temp.getStar();
                if(temp.getStar()) {
                    ((Button)findViewById(R.id.star)).setBackgroundResource(R.drawable.full_star);
                }
                else ((Button)findViewById(R.id.star)).setBackgroundResource(R.drawable.empty_star);
            ```

- 收藏夹界面的UI设计：
    收藏夹与主页面设计思路基本一致，点击事件与前面主页面的也是一样的，就不再叙述了，不过根据项目要求，收藏夹的界面使用的ListView而不是recyclerView来实现的，大体上使用方法无差，不过ListView对应的adapter`（myListViewAdapter.java）`还是需要去实现的，为了保证后面主页的物品添加到收藏夹中，这里的ListView我依旧使用的是`Collection`作为参数。
    ```xml
        <ListView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:id="@+id/shoucangjia"
        android:visibility="invisible"
        />
    ```
- 详情页的UI设计：
    1. 项目要求顶部栏占整个界面的1/3，这里我使用了`layout_weight`权重属性来分配各部分组件所占的比例。
        ```xml
        <LinearLayout xmlns:tools="http://schemas.android.com/tools"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:weightSum="9"
        ···
        <RelativeLayout
            android:layout_width="fill_parent"
            android:layout_height="0dp"
            android:layout_weight="3"
        ···
        ···
        ```
    2. 顶部由四部分组成，一个View组件调整顶部栏的背景色，一个TextView显示item的名字，两个Button分别是返回和星星按钮，且这四部分组件布局使用的依旧是`RelativeLayout`。
    3. 界面中部我分配的权值为2，上下两部分各占权值1，包含了显示itemType、item的nutrition、分割线以及收藏夹按钮。
        ```xml
        <RelativeLayout
        android:layout_width="fill_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        >
            <TextView
                ···
                android:id="@+id/foodType"
                android:textColor="#D5000000"
                android:layout_marginLeft="15dp"
                android:layout_marginTop="10dp"
                ···
                />
            <TextView
                ···
                android:id="@+id/info"
                ···
                android:layout_marginBottom="10dp"
                android:layout_alignParentBottom="true"
                android:text="富含"
                />
            <TextView
                ···
                android:id="@+id/nutrition"
                ···
                android:text="富含"
                />
            <View
                ···
                android:layout_toLeftOf="@id/collect"
                android:layout_centerVertical="true"/>

            <Button
                ···
                android:id="@+id/collect"
                ···
                android:background="@drawable/collect"
                />
            ····
        </RelativeLayout>
        ```
    4. 最后一部分底部的“更多信息”使用的是简答的Listview实现，不做过多赘述。
        ```xml
        <ListView
        android:layout_width="fill_parent"
        android:layout_height="0dp"
        android:id="@+id/moreInfo"
        android:layout_weight="4"
        />
        ```
    ※※※※※
    5. （重点）详情页部分的重点，有两个。
    - 详情页需要获取主页点击的Collection参数并使用其来进行详情页布局，这部分在上面主页的点击事件中已经进行叙述。
    - 在详情页点击`<`之后我们需要返回至主页，而我们在详情页面点击了星星和收藏之后这个信息，又是是要作为参数同时返回给主页的，然后在主页中把状态保存下来，这里依旧是通过`Intent与bundle`来进行参数的传递。
        - 详情页参数传出：
            ```java
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(detailActivity.this , foodListMainActivity.class);
                if(v.getId()==R.id.return_button){
                    backFoodItem= ((TextView)findViewById(R.id.foodName)).getText().toString();
                    intent.putExtra("backFoodItem",  backFoodItem);
                    intent.putExtra("collect", clickCollectFlag);
                    intent.putExtra("star",starFlag);
                    setResult(1,intent);
                    finish();
                }
                else if(v.getId()==R.id.star){
                    temp.setStar(!temp.getStar());
                    starFlag=!starFlag;
                    if(temp.getStar()) {
                        ((Button)findViewById(R.id.star)).setBackgroundResource(R.drawable.full_star);
                    }
                    else ((Button)findViewById(R.id.star)).setBackgroundResource(R.drawable.empty_star);
                }
                else if(v.getId()==R.id.collect){
                    toast= Toast.makeText(getApplicationContext(),"已收藏", Toast.LENGTH_SHORT);
                    toast.show();
                    clickCollectFlag=true;
                }
            }
            ```
            - 主页接收并使用参数——`onActivityResult`函数：
                ```java
                //点击收藏后返回时给收藏夹添加新物品
                //本次实验最难点
                @Override
                protected void onActivityResult(int requestCode, int resultCode, Intent data) {
                    super.onActivityResult(requestCode, resultCode, data);
                    backFoodItem=data.getStringExtra("backFoodItem");
                    for(int i=0;i < this.data.size();i++){
                        if(this.data.get(i).getName().equals(backFoodItem)){
                            if(data.getBooleanExtra("collect",false))
                                this.data2.add(this.data.get(i));
                            this.data.get(i).setStar(data.getBooleanExtra("star",false));
                            myListViewAdapter.notifyDataSetChanged();
                            break;
                        }
                    }
                ```


- 去掉标题栏：在文件(AndroidManifest.xml)中定义项目的主题
    ```xml
    android:theme="@style/AppTheme"
    ```

### (3)本次项目所做加分项

1. 星星图标状态存储——这部分在上面已经提及到了，主要是在页面切换时把星星状态也作为一个参数传回主页并存到相应的Collection中，在每次进入Collection的详情页时都做如下判断即可。。
    ```java
    starFlag = temp.getStar();
        if(temp.getStar()) {
            ((Button)findViewById(R.id.star)).setBackgroundResource(R.drawable.full_star);
        }
        else ((Button)findViewById(R.id.star)).setBackgroundResource(R.drawable.empty_star);
    ```

2. 结合前两周的任务，把之前的“健康服务平台”与本次实验结合起来，在搜索框中输入“FoodList”，点击搜索，即可跳转至本次的foodList实验
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_加分项1.png)

    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_加分项3.png)

    ```java
    intent.setClass(MainActivity.this,foodListMainActivity.class);
        search_button.setOnClickListener(new View.OnClickListener(){
            public void onClick (View v){
                String editString=editText.getText().toString();
                if(editString.length()==0){
                   ···
                }
                else if("Health".equals(editString))
                    ···
                //跳转至foodList
                else if("FoodList".equals(editString))
                    startActivity(intent);
    ```

3. recyclerView加载时添加载入动画，下拉特效，比较酷炫。
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_加分项2.png)
    ![](/img/in-post/post-Android/lab1/截图/lab1_week6_加分项4.png)
    ```xml
    //layout_anim.xml
    <?xml version="1.0" encoding="utf-8"?>
    <layoutAnimation
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:animation="@anim/item_animation"
        android:delay="15%"
        android:animationOrder="normal"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    ```
    ```xml
    //item_animation.xml
    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="wrap_content"
        android:duration="@android:integer/config_mediumAnimTime" android:layout_height="wrap_content">
        <translate
            android:fromYDelta="-20%"
            android:toYDelta="0"
            android:interpolator="@android:anim/decelerate_interpolator"
            />
        <alpha
            android:fromAlpha="0"
            android:toAlpha="1"
            android:interpolator="@android:anim/decelerate_interpolator"
            />
        ···
        ···
    </set>
    ```
    



### (4)实验遇到的困难以及解决思路

这次基本事件处理的实验遇到的主要问题有两个：

1. 收藏夹出来的UI并没有按我想象中出现，如下，该有的margin并没有出来，但是我对收藏夹的处理是和主页的Collection是一样的，这就很苦恼了，后续对比主页和收藏页的Collection，发现是在xml布局文件中有不同，修改后便回到正常了。

![](/img/in-post/post-Android/lab1/截图/lab1_week6_问题1.png)
    
2. 在详情页返回至主页的过程中，我遇到了一个大问题，就是参数无论如何都没办法传回给主页，但是AS的debug还没开始研究，这时候我就用了一个方法来debug，在传参的过程里用`System.out.println(xx)`函数，通过看有无输出来判断某步语句是否执行，最终找到了一个问题:
    ```java
    for(int i=0;i < this.data.size();i++){
                if(this.data.get(i).getName().equals(backFoodItem)){
        ···
    ```                 
    就是这里的equals，之前使用的是连等号，java中连等号只能判断数字是否相等，在判断字符串是否相等时,连等号判断的是地址是否相等，所以是无效的，参数其实是传回给主页了，但是在使用的时候判断出错所以没有使用到。这个问题在之前也遇到过了，C++里的代码习惯还是影响到我在java中的学习啊。



---

## 四、实验思考及感想

实验中的思考与感悟：

- 这次任务是主要考察`Intent、Bundle`的使用，来处理页面切换时的参数传达问题，这次自己是花了很多的时间来阅读相关文档资料，才熟悉好`Intent、Bundle`的使用，虽然使用过程中遇到了小问题，但也算是给我提了个醒，对星星状态的保存也算是我对intent的一个灵活使用。

- 这次实验我使用了一个野蛮的debug方法——`System.out.println(xx)`，虽然技术含量不高，但是也让我在以后遇到问题时能多种角度去思考。
- 加分项中我第一次接触了安卓中的动画，感觉相对比较好理解，简单好用，同时也能使项目更加酷炫，激发了我后续继续学习安卓的动力，希望后面再做出更好的动画效果。这次实验花费许多时间，但是学到的知识点也非常多，收获满满。 

---

