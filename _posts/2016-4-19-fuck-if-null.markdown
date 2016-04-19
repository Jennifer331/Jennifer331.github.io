---
layout: post
title: "if (null == fuck)"
---
# <font color="#e37964">为了不挂</font>
我表示对公司测试的标准很不满意，一些没有必要考虑的用户场景，诶~

```JAVA

Map<String, Bitmap> result = new HashMap<String, Bitmap>();
PackageManager pm = context.getPackageManager();
if(null != pm){
    try{
        Resources res = pm.getResourcesForApplication(packageName);
        if(null != res){
            for(String item : drawableNames){
                int resId = res.getIdentifier(item, "drawable", packageName);
                Bitmap bitmap = BitmapFactory.decodeResource(res, resId);
                if(null != bitmap){
                    result.put(item, bitmap);
                }
            }
        }
    }catch(NameNotFoundException e){
        return null;
    }
}
return result;

```

其实吧，我就是感觉判断这么多空指针，代码看起来糟透了。

# <font color="#e37964">我喜欢简洁利索的代码</font>

```JAVA

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

```

# <font color="#e37964">if-else与try-catch的思辨</font>

可是写出来真的好害怕。。。如果有变态想方设法地把我要请求资源的这个内置应用卸载了的话，这里真的是会抛空指针异常的。。。可是真的不想写丑代码怎么办。。。中午等公交的时候，想到了一个good idea。

```JAVA

public static Map<String, Bitmap> getPackageBitmaps(String packageName, Context context, String[] drawableNames){
    Map<String, Bitmap> result = new HashMap<String, Bitmap>();
    try{
        PackageManager pm = context.getPackageManager();
        Resources res = pm.getResourcesForApplication(packageName);
        for(String item : drawableNames){
            int resId = res.getIdentifier(item, "drawable", packageName);
            Bitmap bitmap = BitmapFactory.decodeResource(res, resId);
            result.put(item, bitmap);
        }
        return result;
    }catch(NameNotFoundException e){
        //If we want to do sth
        //eg,show toast
        return null;
    }catch(Exception e){
        Debugger.w(TAG, "[getPackageBitmap]", e);
        return null;
    }
}

```

我想到了try \ catch,如果我们对于异常的处理都一样，那不妨就用try \ catch把它包住好了，让它抛异常，我接着就是了。对~__其实if \ else不应该被滥用的，我想异常并不属于一种逻辑分支，本来就不应该用if \ else来控制__  

# <font color="#e37964">来自同事的新视角</font>
我们知道的异常，有一些是可控的，还存在很多不可控的异常，如果全部catch住，加log，如果需要靠分析log解bug就爽了
