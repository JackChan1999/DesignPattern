## 1. 模式的定义

简单工厂模式又称为静态方法工厂模式，是由一个工厂对象决定创建哪一个产品类的实例。

## 2. 使用场景

客户端需要创建对象、隐藏对象的创建过程，且目标对象类型数量不多的情况下，可以考虑使用简单工厂模式。

## 3. UML类图

![](http://img.blog.csdn.net/20140417161708078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmJveWZlaXl1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 4. 角色介绍

- Product：产品的通用接口，定义产品的行为
- ConcreteProduct：具体产品类，实现了Product接口
- Creator：工厂类，通过静态工厂方法factory来创建对象

## 5. 简单示例

下面以一个简单的示例来说明进一步理解上述的几个角色。

苹果公司的Iphone系列产品是自从上市以来受到大家的普遍欢迎，但是对于程序员来说，创建一个Iphone产品对象的过程是复杂的，而普通的客户程序并不需要知道这个复杂的创建过程。客户程序只需要调用某个创建方法，就希望得到他想要的对象。如果产品类型不多，我们可以通过简单工厂方法模式来实现产品创建过程的封装。

```java
package com.dp.example.simplefactory;  

/**
 * Iphone产品类, 对应Product角色
 * @author mrsimple
 *
 */  
public interface Iphone {  
    /**
     * 打电话
     */  
    public void call();  

    /**
     * 发短信
     */  
    public void sendMessage();  

    /**
     * 上网
     */  
    public void surfTheInternet();  
}  

package com.dp.example.simplefactory;  

/**
 * Iphone5s 类,对应ConcreteProduct角色
 *  
 * @author mrsimple
 *
 */  
public class Iphone4s implements Iphone {  

    @Override  
    public void call() {  
        System.out.println("用Iphone4s打电话");  
    }  

    @Override  
    public void sendMessage() {  
        System.out.println("用Iphone4s发短信");  
    }  

    @Override  
    public void surfTheInternet() {  
        System.out.println("用Iphone4s上网");  
    }  

}  

package com.dp.example.simplefactory;  

/**
 * Iphone5s 类,对应ConcreteProduct角色
 *  
 * @author mrsimple
 *
 */  
public class Iphone5s implements Iphone {  

    @Override  
    public void call() {  
        System.out.println("用Iphone5s打电话");  
    }  

    @Override  
    public void sendMessage() {  
        System.out.println("用Iphone5s发短信");  
    }  

    @Override  
    public void surfTheInternet() {  
        System.out.println("用Iphone5s上网");  
    }  

}  

package com.dp.example.simplefactory;  

/**
 * 工厂类
 *  
 * @author mrsimple
 *
 */  
public final class IphoneFactory {  
    /**
     * 创建产品
     *  
     * @param type
     * @return
     */  
    public static Iphone createIphone(String type) {  
        if (type == null) {  
            return null;  
        }  

        Iphone iphone = null;  
        if (type.equals("5s")) {  
            iphone = new Iphone5s();  
        } else if (type.equals("4s")) {  
            iphone = new Iphone4s();  
        }  

        return iphone;  
    }  
}
```

使用实例

```java
public static void main(String[] args) {  
    Iphone iphone = IphoneFactory.createIphone("4s") ;  
    iphone.sendMessage();  
    iphone.surfTheInternet();  

    iphone = IphoneFactory.createIphone("5s") ;  
    iphone.sendMessage();  
    iphone.surfTheInternet();  
}
```
输出结果

![](http://img.blog.csdn.net/20140417163806406?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmJveWZlaXl1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 6. 源码分析

在Android中，我们经常使用静态工厂方法的地方应该是创建Bitmap对象的时候，例如通过资源id获取Bitmap对象。

```java
Bitmap bmp = BitmapFactory.decodeResource(getResources(), R.drawable.ic_launcher) ;  
```

那么我们看看BitmapFactory的工厂方法具体实现。

```java
/**
 * Synonym for {@link #decodeResource(Resources, int, android.graphics.BitmapFactory.Options)}
 * will null Options.
 *
 * @param res The resources object containing the image data
 * @param id The resource id of the image data
 * @return The decoded bitmap, or null if the image could not be decode.
 */  
public static Bitmap decodeResource(Resources res, int id) {  
    return decodeResource(res, id, null);  
}  

**  
 * Synonym for opening the given resource and calling  
 * {@link #decodeResourceStream}.  
 *  
 * @param res   The resources object containing the image data  
 * @param id The resource id of the image data  
 * @param opts null-ok; Options that control downsampling and whether the  
 *             image should be completely decoded, or just is size returned.  
 * @return The decoded bitmap, or null if the image data could not be  
 *         decoded, or, if opts is non-null, if opts requested only the  
 *         size be returned (in opts.outWidth and opts.outHeight)  
 */  
public static Bitmap decodeResource(Resources res, int id, Options opts) {  
    Bitmap bm = null;  
    InputStream is = null;   

    try {  
        final TypedValue value = new TypedValue();  
        is = res.openRawResource(id, value);  

        bm = decodeResourceStream(res, value, is, null, opts);  
    } catch (Exception e) {  
        /*  do nothing.
            If the exception happened on open, bm will be null.
            If it happened on close, bm is still valid.
        */  
    } finally {  
        try {  
            if (is != null) is.close();  
        } catch (IOException e) {  
            // Ignore  
        }  
    }  

    if (bm == null && opts != null && opts.inBitmap != null) {  
        throw new IllegalArgumentException("Problem decoding into existing bitmap");  
    }  

    return bm;  
}  
```
可以看到，decodeResource(Resourcesres,intid)函数调用的是decodeResource(Resourcesres,intid, Options opts), 在decodeResource函数中，最终把传递进来的资源id解析成InputStream, 然后称调用decodeResourceStream(res, value, is, null,opts)方法。再看decodeResourceStream的实现。

```java
/**
 * Decode a new Bitmap from an InputStream. This InputStream was obtained from
 * resources, which we pass to be able to scale the bitmap accordingly.
 */  
public static Bitmap decodeResourceStream(Resources res, TypedValue value,  
        InputStream is, Rect pad, Options opts) {  

    if (opts == null) {  
        opts = new Options();  
    }  

    if (opts.inDensity == 0 && value != null) {  
        final int density = value.density;  
        if (density == TypedValue.DENSITY_DEFAULT) {  
            opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;  
        } else if (density != TypedValue.DENSITY_NONE) {  
            opts.inDensity = density;  
        }  
    }  

    if (opts.inTargetDensity == 0 && res != null) {  
        opts.inTargetDensity = res.getDisplayMetrics().densityDpi;  
    }  

    return decodeStream(is, pad, opts);  
}  

/**
 * Decode an input stream into a bitmap. If the input stream is null, or
 * cannot be used to decode a bitmap, the function returns null.
 * The stream's position will be where ever it was after the encoded data
 * was read.
 *
 * @param is The input stream that holds the raw data to be decoded into a
 *           bitmap.
 * @param outPadding If not null, return the padding rect for the bitmap if
 *                   it exists, otherwise set padding to [-1,-1,-1,-1]. If
 *                   no bitmap is returned (null) then padding is
 *                   unchanged.
 * @param opts null-ok; Options that control downsampling and whether the
 *             image should be completely decoded, or just is size returned.
 * @return The decoded bitmap, or null if the image data could not be
 *         decoded, or, if opts is non-null, if opts requested only the
 *         size be returned (in opts.outWidth and opts.outHeight)
 */  
public static Bitmap decodeStream(InputStream is, Rect outPadding, Options opts) {  
    // we don't throw in this case, thus allowing the caller to only check  
    // the cache, and not force the image to be decoded.  
    if (is == null) {  
        return null;  
    }  

    // we need mark/reset to work properly  

    if (!is.markSupported()) {  
        is = new BufferedInputStream(is, 16 * 1024);  
    }  

    // so we can call reset() if a given codec gives up after reading up to  
    // this many bytes. FIXME: need to find out from the codecs what this  
    // value should be.  
    is.mark(1024);  

    Bitmap  bm;  

    if (is instanceof AssetManager.AssetInputStream) {  
        bm = nativeDecodeAsset(((AssetManager.AssetInputStream) is).getAssetInt(),  
                outPadding, opts);  
    } else {  
        // pass some temp storage down to the native code. 1024 is made up,  
        // but should be large enough to avoid too many small calls back  
        // into is.read(...) This number is not related to the value passed  
        // to mark(...) above.  
        byte [] tempStorage = null;  
        if (opts != null) tempStorage = opts.inTempStorage;  
        if (tempStorage == null) tempStorage = new byte[16 * 1024];  
        bm = nativeDecodeStream(is, tempStorage, outPadding, opts);  
    }  
    if (bm == null && opts != null && opts.inBitmap != null) {  
        throw new IllegalArgumentException("Problem decoding into existing bitmap");  
    }  

    return finishDecode(bm, outPadding, opts);  
}  

private static Bitmap finishDecode(Bitmap bm, Rect outPadding, Options opts) {  
    if (bm == null || opts == null) {  
        return bm;  
    }  

    final int density = opts.inDensity;  
    if (density == 0) {  
        return bm;  
    }  

    bm.setDensity(density);  
    final int targetDensity = opts.inTargetDensity;  
    if (targetDensity == 0 || density == targetDensity || density == opts.inScreenDensity) {  
        return bm;  
    }  

    byte[] np = bm.getNinePatchChunk();  
    final boolean isNinePatch = np != null && NinePatch.isNinePatchChunk(np);  
    if (opts.inScaled || isNinePatch) {  
        float scale = targetDensity / (float)density;  
        // TODO: This is very inefficient and should be done in native by Skia  
        final Bitmap oldBitmap = bm;  
        bm = Bitmap.createScaledBitmap(oldBitmap, (int) (bm.getWidth() * scale + 0.5f),  
                (int) (bm.getHeight() * scale + 0.5f), true);  
        oldBitmap.recycle();  

        if (isNinePatch) {  
            np = nativeScaleNinePatch(np, scale, outPadding);  
            bm.setNinePatchChunk(np);  
        }  
        bm.setDensity(targetDensity);  
    }  

    return bm;  
}  
```

decodeResourceStream初始化一些配置和像素信息以后，调用decodeStream(is,pad,opts)，最终调用nativeDecodeAsset或者nativeDecodeStream来构建Bitmap对象，这两个都是native方法( Android中使用Skia库来解析图像 )，本文不进行讨论。最后调用finishDecode函数来设置像素、配置信息、缩放等参数，最终返回Bitmap对象。

BitmapFactory中的decodeFile、decodeByteArray工厂方法都是这么一个类似的过程，BitmapFactory通过不同的工厂方法与传递不同的参数调用不同的图像解析函数来构造Bitmap对象，与上文IphoneFactory通过不同类型参数构造不同的Iphone手机思路相似。

## 7. 优点和缺点

优点：

- 分工明确，各司其职；
- 客户端不再创建对象，而是把创建对象的职责交给了具体的工厂去创建；
- 使抽象与实现分离, 客户程序不知道具体实现；
- 具名工厂函数，更能体现出代码含义。

缺点：

- 工厂的静态方法无法被继承；
- 代码维护不易，对象要是很多的话，工厂是一个很庞大的类；
- 违反开闭原则，如果有新的产品加入到系统中就要修改工厂类。
