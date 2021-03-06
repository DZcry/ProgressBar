# ProgressBar

>在网上翻阅了很多关于ProgressBar滚动效果，但是始终没有找到适合项目中的这种效果，故自己写篇博文，记录一下写作过程，给大家做一个参考。先看下最终效果效果图

![这里写图片描述](http://img.blog.csdn.net/20170907172219783?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzY2Njg3MzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>我这里用的是LICEcap软件录制的gif图，效果有点掉帧，哪位仁兄有比较好的录制gif的软件烦请相告，小弟在此先行谢过。

首先看下xml代码，只有两个系统控件，一个TextView和一个ProgressBar，Button只是为了方便触发进度条的效果，实际项目中可以根据需求来做。首先看下xml中的代码：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:layout_margin="5dp">

    <TextView
        android:layout_marginTop="50dp"
        android:id="@+id/progress_precent"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="0%"
        android:textColor="#ff00"
        android:textSize="12sp"
        android:padding="3dp"/>

    <ProgressBar
        android:id="@+id/pb_progressbar"
        style="?android:attr/progressBarStyleHorizontal"
        android:layout_width="match_parent"
        android:layout_height="5dp"
        android:max="100"
        android:progress="0"
        android:progressDrawable="@drawable/progressbar_color" />

    <Button
        android:id="@+id/btn_start"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="30dp"
        android:text="start" />
</LinearLayout>
```

**样式：progressbar_color.xml**

```
    <?xml version="1.0" encoding="utf-8"?>
	<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >

    <!-- 背景  gradient是渐变,corners定义的是圆角 -->
    <item android:id="@android:id/background">
        <shape>
            <corners android:radius="5dp" />
            <solid android:color="#9a9a9a" />
        </shape>
    </item>
    <!-- 进度条 -->
    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <corners android:radius="5dp" />
                <solid android:color="#E14f50" />
            </shape>
        </clip>
    </item>

	</layer-list>
```

在onCreate()方法中得到控件的宽度，代码如下：

```
        // 得到progressBar控件的宽度
        ViewTreeObserver vto2 = pbProgressbar.getViewTreeObserver();
        vto2.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                pbProgressbar.getViewTreeObserver().removeGlobalOnLayoutListener(this);
                width = pbProgressbar.getWidth();
                Log.i("TAG", "MainActivity onCreate()=="+pbProgressbar.getWidth());
            }
        });
```

先说下具体实现思路，再贴上最终代码

**实现思路是我们需要ProgressBar达到最大值时，TextView的位置移动到最远处。要想得到progressBar的任意百分比可以移动到对应百分比的位置，咱们就需要知道每一个百分比移动的距离。说的还不够明白的话咱们看看下面的公式，可以更好的理解。**

```
	// 进度条的最小单位，默认是1，你也可以是其他数值，我在demo中为了方便使用了1：
	进度条的最小单位 / 进度条的最大值 = 每一个百分比移动的距离/总的距离（控件的总宽度）
	可以推导出：
	每一个百分比要移动距离 = （进度条的最小单位 / 进度条的最大值）*总的距离（控件的总宽度）
```
因为要做移动动画效果，咱们为了避免ANR，直接开一个分线程来控制界面，主要代码如下

```
        //开启分线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                //每一段要移动的距离
                scrollDistance = (float) ((1.0 / pbProgressbar.getMax()) * width);
                for (int i = 1; i <= status; i++) {
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            currentStatue++;
                            currentPosition += scrollDistance;
                            pbProgressbar.incrementProgressBy(1);
                            //做一个平移动画的效果
                            progressPrecent.setTranslationX(currentPosition);
                            progressPrecent.setText(currentStatue + "%");
                        }
                    });
                    try {
                        Thread.sleep(80);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
```
大功告成，但是当我们运行起来的时候发现，当ProgressBar达到最大值时，上面的字体超出了屏幕范围而看不到了。效果如图所示：

![这里写图片描述](http://img.blog.csdn.net/20170907173312004?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzY2Njg3MzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这给使用者造成了很大的困惑，静下心来分析一下，可以知道TextView一直在对应ProgressBar数据的右面，语言功底不太好，咱们上图看：

![这里写图片描述](http://img.blog.csdn.net/20170907173537058?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzY2Njg3MzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看出，当数据为0%时就在progressBar对应数据的右方，当数据为最大值时，超出屏幕显示不了就不足为其了。咱们现在如果想让progressBar是最大值时还能显示，就需要当偏移的距离加上字体的宽度和字体右面的Padding值大于progressBar宽度的时候不偏移。接着修改代码如下：

```
 //开启分线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                //每一段要移动的距离
                scrollDistance = (float) ((1.0 / pbProgressbar.getMax()) * width);
                for (int i = 0; i < status; i++) {
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            // 控制进度条的增长进度
                            pbProgressbar.incrementProgressBy(1);
                            currentStatue++;
                            tvPrecent.setText(currentStatue + "%");
                            // 得到字体的宽度
                            tvWidth = tvPrecent.getWidth();
                            currentPosition += scrollDistance;
                            //做一个平移动画的效果
                            // 这里加入条件判断
                            if (tvWidth + currentPosition <= width - tvPrecent.getPaddingRight()) {
                                tvPrecent.setTranslationX(currentPosition);
                            }
                        }
                    });
                    try {
                        Thread.sleep(80);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
```
我们运行起来，再来看下效果：

![这里写图片描述](http://img.blog.csdn.net/20170907174804312?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMzY2Njg3MzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

到这里咱们就完成了，有不清楚或者写错的地方欢迎留言指正，我会第一时间答复。如果觉得对你有帮助，欢迎star和fork。