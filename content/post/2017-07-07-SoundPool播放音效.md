---
title: "SoundPool播放音效"
date: 2017-07-07
tags: ["Android"]
categories: ["技术之美"]
---
实现Android App 的简单音效
<!-- more --> 

如果应用程序经常播放密集、急促而又短暂的音效(如游戏音效)那么使用Media Player显得有些不太适合了。因为Media Player存在如下缺点：

1. 延时时间较长，且资源占用率高。
2. 不支持多个音频同时播放。

但是，Android中除了Media Player播放音频之外还提供了Sound Pool来播放音效，Sound Pool使用音效池的概念来管理多个短促的音效，例如他可以开始就加载20个音效，以后再程序中按音效的ID进行播放。

```java
SoundPool soundpool = new SoundPool(int maxStreams, int streamType,int srcQuality)
```

| 参数         | 含义         |
| ---------- | ---------- |
| maxStream  | 指定支持多少个声音； |
| streamType | 指定声音类型；    |
| srcQuality | 指定声音品质；    |

得到SoundPool对象之后，接下来就可以调用SoundPool的多个重载的load方法来加载声音了。提供了以下四种load方法：

1. int load(Context context, int resld, int priority)：从 resld 所对应的资源加载声音。
2. int load(FileDescriptor fd, long offset, long length, int priority)：加载 fd 所对应的文件的offset开始、长度为length的声音。
3. int load(AssetFileDescriptor afd, int priority)：从afd 所对应的文件中加载声音。
4. int load(String path, int priority)：从path 对应的文件去加载声音。

加载完之后，我们开始播放音乐，播放音乐的方法：

```java
int play(int soundID, float leftVolume, int rightVolume, int Priority, int loop, float rate)
```

| 参数                     | 含义                          |
| ---------------------- | --------------------------- |
| soundID                | 第一个参数指定播放哪个声音；              |
| leftVolume,rightVolume | 指定左右的音量；                    |
| priority               | 制定播放声音的优先级，数值越大，优先级越高；      |
| loop                   | 指定是否**循环**，0为不循环，-1为循环；     |
| rate                   | 指定播放的比率不，数值可以从0.5-2，1为正常比率。 |

为了更好的管理SoundPool所加载的每个声音的ID，程序一般会使用一个HashMap<Integer, Integer>对象来管理声音。

### 归纳起来，使用SoundPool播放声音的步骤如下：

1. 调用SoundPool的构造器创建SoundPool的对象。
2. 调用SoundPool对象的load()方法从指定资源、文件中加载声音。最好使用HashMap<Integer, Integer>来管理所加载的声音。
3. 调用Sound Pool的play方法播放音效。

```java
public class SoundPoolDeme extends Activity{
  Button btn1,btn2,btn3;
  //创建一个SoundPool对象
    SoundPool soundPool;
    //定义一个HashMap用于存放音频流的ID
    HashMap<Integer, Integer>musicId=new HashMap<Integer, Integer>();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.mian);
       btn1=(Button)findViewById(R.id.btn1);
       btn2=(Button)findViewById(R.id.btn2);
       btn3=(Button)findViewById(R.id.btn3);
       //初始化soundPool,设置可容纳12个音频流，音频流的质量为5，
       soundPool=new SoundPool(12, 0,5);
       //通过load方法加载指定音频流，并将返回的音频ID放入musicId中
       musicId.put(1, soundPool.load(this, R.raw.awooga, 1));
       musicId.put(2, soundPool.load(this, R.raw.evillaugh, 1));
       musicId.put(3, soundPool.load(this, R.raw.jackinthebox, 1));
       OnClickListener listener=new OnClickListener() {       
           @Override
           public void onClick(View v) {
              // TODO Auto-generated method stub
              switch (v.getId()) {
              case R.id.btn1:
                  //播放指定的音频流
                  soundPool.play(musicId.get(1),1,1, 0, 0, 1);
                  break;
              case R.id.btn2:
                  soundPool.play(musicId.get(2),1,1, 0, 0, 1);
                  break;
              case R.id.btn3:
                  soundPool.play(musicId.get(3),1,1, 0, 0, 1);
                  break;
              default:
                  break;
              }
           }
       };
       btn1.setOnClickListener(listener);
       btn2.setOnClickListener(listener);
       btn3.setOnClickListener(listener);
    }
}
```

### 注意：

SoundPool虽然可以一次性加载多个声音，但是由于内存的限制，因此应该避免使用SoundPool来播放歌曲或者做游戏背景音乐，只有那些短促、密集的声音才考虑使用SoundPool进行播放。

虽然SoundPool比MediaPlayer的效果好，但也不是绝对不存在延迟问题，尤其在那些性能不太好的手机中，SoundPool的延迟问题会更加的严重。