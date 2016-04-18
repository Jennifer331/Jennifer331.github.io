---
layout: post
title: "读取其他apk的资源、解bug的心态"
lang: zh
tags: [Android]
---

# <font color="#e3796b">用一个apk共享资源</font>
我们的apk有两张图一向都是另一个团队按需求提供，他们要换，就拿图来，我们换进我们的apk，但是种小事真的是说烦不烦，但是做起来真的感觉好蠢。他们搞出个神奇的apk，一行代码没有，只有资源，然后我们就去读那里的资源就好。

感觉很新奇，以前自己从来没有想到过这种资源共享的方式。但是不知道从安全或者其他的角度考虑会怎么样？我自己还没有想到什么。

```JAVA

/**
 * get Bitmaps from package
 * @param packageName The full name (i.e. com.google.apps.contacts) of the
 *            desired package.
 * @param context
 * @param drawableNames
 * @return
 */
public static Map<String, Bitmap> getPackageBitmaps(String packageName, Context context, String[] drawableNames){
    Map<String, Bitmap> result = new HashMap<String, Bitmap>();
    PackageManager pm = context.getPackageManager();
    try{
        Resources res = pm.getResourcesForApplication(packageName);
        for(String item : drawableNames){
            int resId = res.getIdentifier(item, "drawable", packageName);
            Bitmap bitmap = BitmapFactory.decodeResource(res, resId);
            result.put(item, bitmap);
        }
    }catch(NameNotFoundException e){
        return null;
    }
    return result;
}

```

# <font color="#e3796b">ImageView的setImageBitmap()和setImageDrawable()</font>
从代码看，好像一点也没有区别~

```JAVA

/**
 * Sets a Bitmap as the content of this ImageView.
 *
 * @param bm The bitmap to set
 */
@android.view.RemotableViewMethod
public void setImageBitmap(Bitmap bm) {
    // if this is used frequently, may handle bitmaps explicitly
    // to reduce the intermediate drawable object
    setImageDrawable(new BitmapDrawable(mContext.getResources(), bm));
}

```

# <font color="#e3796b">Bitmap的createBitmap()返回的不一定是一个新的对象</font>
我一直以为这个方法是用来复制一份当前bitmap

```JAVA

/**
 * Returns an immutable bitmap from the source bitmap. The new bitmap may
 * be the same object as source, or a copy may have been made.  It is
 * initialized with the same density as the original bitmap.
 */

```

 从这段注释可以看出来，这个方法返回的是same或者copy,如果目的是要复制一份bitmap的话，应该用 __copy()__  函数

# <font color="#e3796b">解bug的态度</font>
如果你总把bug看成别人的错误，每查到一处问题代码，第一反应不是分析逻辑问题，而是找作者，如果作者在，马上做要甩包袱的姿态，如果作者不在，“艹，又是**挖的坑”。

如果是这种态度，那么别人挖的坑足够把你埋起来。

反过来，如果乐在其中，慢慢地，别人的代码也会变成自己的代码，越来越得心应手。

解bug其实是有趣的事情，用现象去找逻辑，用逻辑去推测现象，要是有心情的时候，看着代码和bug还可以猜得到测试的癖好，哈哈~

# <font color="#e3796b">感觉要被我写成日记了，哈哈</font>
