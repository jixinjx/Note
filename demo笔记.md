

## git版本控制

1.去github上创建自己的Repository

2.在本地文件夹的 git bash中

```
git init
```

3.添加git仓库

```
git add .
```

4.提交到仓库

```
git commit -m "注释语句"
```

5.将本地的仓库关联到GitHub，后面的https改成刚刚自己的地址，上面的红框处

```
git remote add origin https://github.com/zlxzlxzlx/Test.git
```

6.上传之前pull一下

```
git pull origin master
```

7.上传代码到GitHub远程仓库

```
git push -u origin master
```





更新代码

1.更新全部

```
git add *
```

2.接着输入git commit -m "更新说明"

```
git commit -m "更新说明"
```

3.先git pull ,拉去最新分支

```
git pull
```

4.push

```
git push origin master
```



## 实现链路



## 歌曲列表

### 权限获取

权限获取，我们要做的第一件事就是去内存读取音乐相关信息，那么我们就需要获得相关的权限，从Android6.0开始部分权限不仅需要在AndroidManifest.xml文件中声明，还需要在运行程序的时候动态获取.这里以读取存储权限为例：

首先在AndroidManifest.xml中定义

```java
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

在Java业务代码中先判断是否已经有权限，有权限就不再申请，没有就申请权限

```java
//首先检查自身是否已经拥有相关权限，拥有则不再重复申请
int check = checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE) ;
    //没有相关权限
    if (check != PackageManager.PERMISSION_GRANTED)
    {
        //申请权限  STORGE_REQUEST = 1
        ActivityCompat.requestPermissions(this , new String[] {Manifest.permission.WRITE_EXTERNAL_STORAGE} ,STORGE_REQUEST);
    }else {
    //已有权限的情况下可以直接初始化程序
    init();
}
```

### 创建页面

需要创建两个xml，首先是listview的xml

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@+id/music_list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

然后定制listview的界面

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="horizontal"
    android:layout_height="match_parent">

    <!--<TextView-->
    <!--android:id="@+id/MusicTitle"-->
    <!--android:layout_width="80dp"-->
    <!--android:layout_height="wrap_content"-->
    <!--android:layout_marginTop="3dp"-->
    <!--android:layout_marginLeft="5dp"-->
    <!--/>-->
    <ImageView
        android:id="@+id/MusicImage"
        android:layout_width="60dp"
        android:layout_height="60dp"
        android:scaleType="fitCenter"
        android:layout_margin="5dp"
        />
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"

        android:layout_height="wrap_content">
        <TextView
            android:id="@+id/MusicName"
            android:layout_width="match_parent"
            android:layout_marginLeft="10dp"
            android:layout_marginTop="3dp"
            android:layout_height="wrap_content"
            android:ellipsize="end"
            android:singleLine="true"
            android:textSize="20dp"
            />
        <TextView
            android:id="@+id/MusicSize"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="3dp"
            android:layout_marginLeft="10dp"
            android:textSize="20dp"
            />
    </LinearLayout>
</LinearLayout>
```

## 创建创建存储音乐信息MusicInfo工具类

```java
package com.example.demo.PackageClass;

import java.io.Serializable;

/**
 * Created by initializing on 2018/4/25.
 音乐信息封装类
 */

public class MusicInfo implements Serializable {
    private  int _id = - 1;       //音乐标识码
    private  int duration = -1 ;   //音乐时长
    private String artist = null ;    //音乐作者
    private String musicName= null ; //音乐名字
    private String album = null ;   //音乐文件专辑
    private String title = null ;  //音乐文件标题
    private int size  ;   //音乐文件的大小  返回byte大小
    private String data ;  //获取文件的完整路径
    private String album_id ; //实际存储为音乐专辑团片



    public String getAlbum_id()
    {
        return album_id ;
    }
    public void setAlbum_id(String album_id)
    {
        this.album_id = album_id ;
    }
    public void setData(String data)
    {
        this.data = data ;
    }
    public String getData()
    {
        return data ;
    }
    public void setSize(int size)
    {
        this.size = size ;
    }
    public int getSize()
    {
        return size ;
    }
    public void setTitle(String title)
    {
        this.title = title ;
    }
    public String getTitle()
    {
        return title ;
    }
    public String getAlbum()
    {
        return album ;
    }
    public void setAlbum(String album)
    {
        this.album = album ;
    }
    public int get_id()
    {
        return _id ;
    }
    public int getDuration()
    {
        return duration ;
    }
    public String getArtist()
    {
        return artist ;
    }
    public String getMusicName()
    {
        return musicName ;
    }
    public void set_id(int _id)
    {
        this._id = _id ;
    }
    public void setDuration(int duration)
    {
        this.duration = duration ;
    }
    public void setArtist(String artist)
    {
        this.artist = artist ;
    }
    public void setMusicName(String musicName)
    {
        this.musicName = musicName ;
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
    }
}
```



## 创建初始化函数、

首先要知道基本数据库查询操作，返回Cursor对象

public final Cursor query(Uri uri , //查询路径

​              String[] projection , //查询指定的列

​              String selection, //查询条件

​              String[] selectionArgs, //查询参数

​              String sortOrder } //查询结果的排序方式



#### SimpleAdapter适配器

```
public SimpleAdapter (Context context, 
                List<? extends Map<String, ?>> data, 
                int resource, 
                String[] from, 
                int[] to)
```

| Parameters |                                                              |
| :--------- | ------------------------------------------------------------ |
| `context`  | `Context`: The context where the View associated with this SimpleAdapter is running |
| `data`     | `List`: A List of Maps. Each entry in the List corresponds to one row in the list. The Maps contain the data for each row, and should include all the entries specified in "from" |
| `resource` | `int`: Resource identifier of a view layout that defines the views for this list item. The layout file should include at least those named views defined in "to" |
| `from`     | `String`: A list of column names that will be added to the Map associated with each item. |
| `to`       | `int`: The views that should display column in the "from" parameter. These should all be TextViews. The first N views in this list are given the values of the first N columns in the from parameter. |

```java
//初始化
    private void init(){
        List_map = new ArrayList<Map<String, String>>() ;
        musicInfos = new ArrayList<>() ;

        try {
            // 查询音乐列表数据
            //获取系统的ContentResolver
            contentResolver = getContentResolver() ;
            //从数据库中获取指定列的信息
            mCursor = contentResolver.query(MediaStore.Audio.Media.EXTERNAL_CONTENT_URI ,
                    new String[] {MediaStore.Audio.Media._ID ,
                            MediaStore.Audio.Media.TITLE , MediaStore.Audio.Media.ALBUM ,
                            MediaStore.Audio.Media.ARTIST , MediaStore.Audio.Media.DURATION ,
                            MediaStore.Audio.Media.DISPLAY_NAME , MediaStore.Audio.Media.SIZE ,
                            MediaStore.Audio.Media.DATA , MediaStore.Audio.Media.ALBUM_ID } , null ,null ,null) ;
            
            if (mCursor != null && mCursor.getCount()>=1) {
                while (mCursor.moveToNext()) {
                    Map<String , String> map = new HashMap<>() ;
                    MusicInfo musicInfo = new MusicInfo() ;
                    //将数据装载到List<MusicInfo>中
                    musicInfo.set_id(mCursor.getInt(0));
                    musicInfo.setTitle(mCursor.getString(1));
                    musicInfo.setAlbum(mCursor.getString(2));
                    musicInfo.setArtist(mCursor.getString(3));
                    musicInfo.setDuration(mCursor.getInt(4));
                    musicInfo.setMusicName(mCursor.getString(5));
                    musicInfo.setSize(mCursor.getInt(6));
                    musicInfo.setData(mCursor.getString(7));
                    //将数据装载到List<Map<String ,String>>中
                    //获取本地音乐专辑图片
                    String MusicImage = getAlbumArt(mCursor.getInt(8)) ;
                    //判断本地专辑的图片是否为空
                    if (MusicImage == null)
                    {
                        //为空，用默认图片
                        map.put("image" , String.valueOf(R.mipmap.timg)) ;
                        musicInfo.setAlbum_id(String.valueOf(R.mipmap.timg));
                    }else
                    {
                        //不为空，设定专辑图片为音乐显示的图片
                        map.put("image" , MusicImage) ;
                        musicInfo.setAlbum_id(MusicImage);
                    }
                    // musicInfo.setAlbum_id(mCursor.getInt(8));
                    musicInfos.add(musicInfo) ;
                    map.put("name" , mCursor.getString(5)) ;
                    //将获取的音乐大小由Byte转换成mb 并且用float个数的数据表示
                    Float size = (float) (mCursor.getInt(6) * 1.0 / 1024 / 1024)  ;
                    //对size这个Float对象进行保留两位小数处理
                    BigDecimal b  =   new  BigDecimal(size);
                    Float   f1   =  b.setScale(2,  BigDecimal.ROUND_HALF_UP).floatValue();
                    map.put("size" , f1.toString() + "mb") ;
                    List_map.add(map) ;
                }
                //adapter.notifyDataSetChanged();
                //SimpleAdapter实例化，读取音乐文件的封面、名字、大小等
                simpleAdapter = new SimpleAdapter(this 
                            , List_map 	                                                  		                      ,R.layout.music_adapte_list_view 
                           ,new String[] {"image" , "name" , "size"} 
                           ,new int[]{R.id.MusicImage , R.id.MusicName , R.id.MusicSize}) ;
               
                //为ListView对象指定adapter
                ListView MusicListView = (ListView) findViewById(R.id.music_list_view);
                MusicListView.setAdapter(simpleAdapter);
                
            }else{
               
            }
        } catch (Exception e) {
            e.printStackTrace();
            
        } finally {
            if (mCursor != null) {
                mCursor.close();

            }
        }
    }

```



## 点击跳转

在歌曲列表activity中

```java
/*
   item点击实现
 */

public void onItemClick(AdapterView<?> parent, View view, int position, long id) {

    String MusicData = musicInfos.get(position).getData();
    Log.d("MainActivity", "onItemClick: "+MusicData);

    //将点击位置传递给播放界面，在播放界面获取相应的音乐信息再播放。
    Bundle bundle = new Bundle() ;
    bundle.putInt("position" , position);
    bundle.putSerializable("musicinfo" , (Serializable) getMusicInfos());
    Intent intent = new Intent() ;
    //绑定需要传递的参数
    intent.putExtras(bundle) ;
    intent.setClass(this ,music_player.class ) ;
    startActivity(intent);
}
```

其中用到Bundle进行音乐歌曲信息的传递

在播放器界面进行设置

```java

            bundle = new Bundle();
            bundle = getIntent().getExtras() ;
            
try{
                List<MusicInfo> musicInfosList = (List<MusicInfo>) bundle.getSerializable("musicinfo");
                position = bundle.getInt("position") ;
           /*
           用输出来检测是否成功获取参数
           */
                Log.d("music_player", "onCreate: "+"position is " + position + "" +
                        "\n MUSICINFOLIST DATA IS  " + musicInfosList.get(position).getData());

            }catch (Exception e)
            {
                System.out.println("获取数据失败");
                e.printStackTrace();
            }
```

## 用handler改变UI

Android的UI也是线程不安全的。也就是说，如果想要更新应用程序 里的UI元素，则必须在主线程中进行，否则就会出现异常。

简单的来说，就是子线程中调用handler来发送message,主线程中的handler接收到message以后，进行UI更改

首先声明一个handler，Handler 类应该应该为static类型，否则有可能造成泄露。

https://zhuanlan.zhihu.com/p/259242282

https://sshpark.github.io/2019/02/11/%E5%9C%A8Android%E4%B8%AD%E6%9B%B4%E6%96%B0UI%E7%9A%84%E5%87%A0%E7%A7%8D%E6%96%B9%E6%B3%95/

在这里的解决方法如下：

```
private Handler music_info_handler = new Handler(new Handler.Callback() {

        @Override
        public boolean  handleMessage(Message msg) {
            switch (msg.what) {
                case UPDATE_TEXT:
// 在这里可以进行UI操作
                    music_title.setText(musicInfosList.get(position).getTitle());
                    music_artist.setText(musicInfosList.get(position).getArtist());
                    break;
                default:
                    break;
            }
        return true;
        }
    });
```

而后创建线程，发送message，同时注意设定message.what的值

```

        new Thread(new Runnable() {
            @Override
            public void run() {
                Message message = new Message();
                message.what = UPDATE_TEXT;
                music_info_handler.sendMessage(message); // 将Message对象发送出去
            }
        }).start();
```

我们并没有在子线程里 直接进行UI操作，而是创建了一个Message （android.os.Message ）对象，并将它的 what 字段的值指定为UPDATE_TEXT ，然后调用Handler 的sendMessage() 方法将这条 Message发送出去。很快，Handler就会收到这条Message，并在handleMessage() 方法中对它 进行处理。注意此时handleMessage() 方法中的代码就是在主线程当中运行的了，所以我们 可以放心地在这里进行UI操作。接下来对Message携带的what 字段的值进行判断，如果等于 UPDATE_TEXT ，就将UI的text进行了修改

## 通过广播实现暂停与播放

首先创建本地广播

```java
//创建本地广播_播放暂停
    class Music_start_pauseReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d("Broadcast", "onReceive: receive");
          if(mediaPlayer.isPlaying()){
              mediaPlayer.pause();
          }
          else {
              mediaPlayer.start();
          }
        }
    }
```

在service里注册Receiver，并创建LocalBroadcastManager

```java
 intentFilter = new IntentFilter();
        intentFilter.addAction("com.example.demo.MUSIC_START_PAUSE");
        music_start_pauseReceiver = new Music_start_pauseReceiver();
	//获取实例
        localBroadcastManager = LocalBroadcastManager.getInstance(this);
        localBroadcastManager.registerReceiver(music_start_pauseReceiver, intentFilter);
        // 注册本地广播监听器
```

在music_player中创建 LocalBroadcastManager

```
 //获取实例
        localBroadcastManager = LocalBroadcastManager.getInstance(this);
```

在点击事件中添加发送本地广播的代码

```java
@Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.music_player_start_pause:
                Intent intent = new Intent("com.example.demo.MUSIC_START_PAUSE");//此处为service中addAction的地方
                        localBroadcastManager.sendBroadcast(intent); // 发送本地广播
                Log.d("Broadcast", "onClick: music_player_start_pause");

        }
```

最后记得在service中ondestory()撤销注册

```
 localBroadcastManager.unregisterReceiver(music_start_pauseReceiver);
```

参考：https://www.jianshu.com/p/a3c03c412e8a

## 计时器

通过计时器获取当前音乐播放进度

采用Handler的postDelayed(Runnable, long) 方法 

```java
 private Handler handler=new Handler();
    Runnable runnable=new Runnable(){
        @Override
        public void run() {
            String show;
            int music_time_tmp=playerService.get_currentduration()/1000;
            if(music_time_tmp%60>=10){
                show=music_time_tmp/60+":"+music_time_tmp%60;
            }else{
                show=music_time_tmp/60+":0"+music_time_tmp%60;
            }

            music_currenttime.setText(show);

            handler.postDelayed(this, 100);
            Log.d("change currentduration", "run: currentduration");
        }
    };
```

启动计时器

```java
handler.postDelayed(runnable, 2000);//每两秒执行一次runnable. 
```

https://blog.51cto.com/u_14397532/3000571

## seekbar相关

首先绑定seekbar组件

```java
 private SeekBar music_seekbar;
 music_seekbar = (SeekBar) findViewById(R.id.seekBar);
```

然后创建seekbarcontrol类来对seekbar进行一些控制

```java
package com.example.demo.PackageClass;

import android.util.Log;

public class SeekBarControl {
    //歌曲时长（毫秒为单位）
    private float max_value=0;
    //当前时间
    private float currentvalue=0;
    //seekbar的最大值
    private int max=0;
    //判断是否在进行seek操作
    private boolean isseeking=false;
    //private int currentvalue=0;

public void SeekBarControl(){

}

    public void setMaxValue(float max_value){
        this.max_value=max_value;
    }

    public void setMinValue(float currentvalue){
        this.currentvalue=currentvalue;
    }
    public void setMax(int max){
        this.max=max;
    }
    //计算应该跳转到的进度条的值
    public int getProgress(){
        int result;
        result =(int) (max*currentvalue/max_value);
        return result;
    }
    //获取当前时间的string(已被替代)
//    public String getCurrentDuration(){
//        int currentvalue_int =(int) currentvalue/1000;
//        int max_value_int =(int) max_value/1000;
//        String show=null;
//        if(currentvalue_int%60>=10){
//            show=currentvalue_int/60+":"+currentvalue_int%60;
//        }else{
//            show=currentvalue_int/60+":0"+currentvalue_int%60;
//        }
//        return show;
//    }
//计算要跳转到的时间（毫秒）
    public int setCurrentDuration(int seek_progress){
    int result;
    result=(int) (seek_progress*max_value/max);
        Log.d("seeking", "seekbarcontrol.setCurrentDuration"+result);
    return result;
    }
    //判断现在是否在seek
    public boolean isSeeking(){
    return this.isseeking;
    }
    //设置seek状态
    public void setSeeking(boolean isseeking){
        this.isseeking=isseeking;
    }
    //获取当前时间
    public String getBarTime(int seek_progress){
        double seek_progress_float =seek_progress*1.0000;

        int seek_progress_int =(int) (seek_progress_float/1000*max_value/1000);

        String show=null;
        if(seek_progress_int%60>=10){
            show=seek_progress_int/60+":"+seek_progress_int%60;
        }else{
            //不足十秒的进行补0
            show=seek_progress_int/60+":0"+seek_progress_int%60;
        }
        Log.d("seeking", "seekbarcontrol.show"+show);
        return show;
    }

}

```

重写seekbar的事件

```java
     music_seekbar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            //触碰上去的时候
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                //playerService.setMediaPlayer(mediaPlayer);
                //改变现在的时间为拖动进度显示的时间
                //music_seekbar.getProgress()获取当前bar的值
                //seekbarcontrol.getBarTime(）将BAR的值转化为这首歌的时间点，返回的为string 如01：23
                music_currenttime.setText(seekbarcontrol.getBarTime(music_seekbar.getProgress()));
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {
            //当开始拖动进度条时，设置为正在拖动
               seekbarcontrol.setSeeking(true);
                Log.d("seeking", "onStartTrackingTouch: ");
            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {

                playerService.setIndex(position);
                playerService.setMusicInfoList(musicInfosList);
                //计算要拖动到的时间位置（毫秒）
                int seek_to=seekbarcontrol.setCurrentDuration(seekBar.getProgress());

                playerService.setDuration(seek_to);
                playerService.setMediaPlayer(mediaPlayer);
                //设置为未在拖动状态
                seekbarcontrol.setSeeking(false);

                Log.d("seeking", "onStopTrackingTouch: ");
            }


        });
```



修改之前的计时器，

```java

    private Handler handler=new Handler();
    Runnable runnable=new Runnable(){
        @Override
        public void run() {
            //String show;
            //判断是否在播放;
            if(playerService !=null){
                seekbarcontrol.setMax(1000);
                //设置最大值
                seekbarcontrol.setMaxValue(mediaPlayer.getDuration());
                //设置正在播放的时间
                seekbarcontrol.setMinValue(playerService.get_currentduration());
                

                Log.d("seeking", "run: before set progress"+"result"+seekbarcontrol.getProgress());
                Log.d("seeking", "run: "+seekbarcontrol.getBarTime(music_seekbar.getProgress()));
                //如果不在进行拖动活动
                if(!seekbarcontrol.isSeeking()){
                    //设置托条进度
                    music_seekbar.setProgress(seekbarcontrol.getProgress());
                    //设置当前时间文本
                    music_currenttime.setText(seekbarcontrol.getBarTime(music_seekbar.getProgress()));
                }



                Log.d("seeking", "run: after set progress"+"result"+seekbarcontrol.getProgress());
                handler.postDelayed(this, 100);
            }

        }
    };

```

## UI改善

### Imagebutton

```
android:scaleType="centerInside" ---图片中间
android:alpha="0.7"   ---透明度
android:background="#00FF0000" ---白色背景
```

### textview

```
android:gravity="center"  --文字居中
```

### 点击改变按钮图片

```
 if(mediaPlayer.isPlaying()){
                    play.setImageResource(R.mipmap.pause);
                }
                else{
                    play.setImageResource(R.mipmap.play);
                }
```

改应用名字

```
应用列表中应用名称在AndroidManifest.xml文件中application标签中的android:label="@string/app_name"设置。

手机桌面上图标下面的名称在AndroidManifest.xml文件中默认activity中的android:label="@string/app_name"设置。
```





bitmap通过resid获取图片

```
 int a =Integer.parseInt(musicInfosList.get(position).getAlbum_id());
                    Bitmap bt_src = BitmapFactory.decodeResource(getResources(),a) ;
```





## 将图片切成圆形

参考：https://www.jianshu.com/p/818e1284158d

## 图片旋转

```java
if(image_rotation<359){
    image_rotation=image_rotation+1;
}else{
    image_rotation=0;
}
music_image.setRotation(image_rotation);
```

调用计时器，通过设置imageview的rotation属性进行选装

