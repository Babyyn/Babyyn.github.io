# drawable资源文件问题汇总

### Android 6 不支持 gradient with offset

```java
Caused by org.xmlpull.v1.XmlPullParserException
Binary XML file line #7: invalid color state list tag gradient

android.content.res.ColorStateList.createFromXmlInner + 217 (ColorStateList.java:217)
android.view.LayoutInflater.createView + 619 (LayoutInflater.java:619)
```

比如下面的一个vector文件，在Android 6 系统中就会导致crash。

```xml
<vector android:height="22dp" android:viewportHeight="156.896"
    android:viewportWidth="156.897" android:width="22dp"
    xmlns:aapt="http://schemas.android.com/aapt" xmlns:android="http://schemas.android.com/apk/res/android">
    <path android:fillColor="#fff" android:pathData="M152.456, 93.734Z"/>
    <path android:pathData="M122.095,0.003L34.79">
        <aapt:attr name="android:fillColor">
            <gradient android:endX="87.7021" android:endY="87.7101"
                android:startX="13.802198" android:startY="13.810199" android:type="linear">
                <item android:color="#FFFF321E" android:offset="0"/>
                <item android:color="#FFED171F" android:offset="1"/>
            </gradient>
        </aapt:attr>
    </path>
</vector>
```

this is because you use a few attributes not supported prior to API level `24`.

eg. `startX`, `endX`, `startY`, `endY` and `offset`.

Android Marshmallow `6.0` is API level `23` ...

that `invalid color state list tag gradient` is coming from the `offset`:

```
<item android:color="#FF2AF598" android:offset="0"/>
<item android:color="#FF009EFD" android:offset="1"/>
```

this issue most likely isn't `LG` specific, but `Marshmallow` specific.

[参考链接](https://stackoverflow.com/questions/53018399/android-invalid-color-state-list-tag-gradient)



### Android 6 AnimatedVectorDrawable NPE

如下一个vector资源文件

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="14dp"
    android:height="14dp"
    android:viewportWidth="24.0"
    android:viewportHeight="24.0">
    <path
        android:fillColor="#797872"
        android:pathData="M11,17h2v-6h-2v6zM12" />
</vector>
```

当使用**AnimatedVectorDrawable**去加载它时，会引发**NullPointerException**。

The crash happens when I try to launch the animation on an `AnimatedVectorDrawable` which has NO `target` attribute (I'm using a [single XML file](https://developer.android.com/reference/android/graphics/drawable/AnimatedVectorDrawable.html) to define the vector drawable and its animations).

More precisely, I think android looks into the `animated-vector` element and throws an error if no `target`  found. 

I can only think of **two solutions** for now:

- either add at least one `target` element into the `AnimatedVectorDrawable`,
- or use a simple `VectorDrawable` if no animation is needed (I used `AnimatedVectorDrawable`regardless whether or not my vector was animated, to be able to easily add animations to it later if I wanted to).

I found a simple workaround to avoid changing everything, that maybe can help others: I just had to add a dumb `target` to all my `AnimatedVectorDrawable` that shouldn't be animated.

```xml
<target android:name="vector">
    <aapt:attr name="android:animation">
        <objectAnimator
            android:propertyName="alpha"
            android:valueTo="1"/>
    </aapt:attr>
</target>
```

修改上面的vector文件：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<animated-vector xmlns:aapt="http://schemas.android.com/aapt"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <aapt:attr name="android:drawable">
        <vector
            android:width="14dp"
            android:height="14dp"
            android:viewportWidth="24.0"
            android:viewportHeight="24.0">
            <path
                android:fillColor="#797872"
                android:pathData="M11,17h2v-6h-2v6zM12" />
        </vector>
    </aapt:attr>
</animated-vector>
```

[参考链接](https://stackoverflow.com/questions/48652597/nullpointerexception-when-starting-animatedvectordrawable-on-api-22)

### vector资源文件转换Bitmap

```java
    public static Bitmap getBitmapFromVectorDrawable(Context context, int drawableId) {
        Drawable drawable = ContextCompat.getDrawable(context, drawableId);
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) {
            drawable = (DrawableCompat.wrap(drawable)).mutate();
        }

        Bitmap bitmap = Bitmap.createBitmap(48, 48, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        drawable.setBounds(0, 0, canvas.getWidth(), canvas.getHeight());
        drawable.draw(canvas);

        try {
            saveBitmap(bitmap, Environment.getExternalStorageDirectory());
        } catch (IOException e) {
            e.printStackTrace();
        }

        return bitmap;
    }

    private static void saveBitmap(Bitmap bitmap, File file) throws IOException {
        //create a file to write bitmap data
        File f = new File(file, "xx.png");
        f.createNewFile();

        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        bitmap.compress(Bitmap.CompressFormat.PNG, 0 /*ignored for PNG*/, bos);
        byte[] bitmapdata = bos.toByteArray();

        FileOutputStream fos = new FileOutputStream(f);
        fos.write(bitmapdata);
        fos.flush();
        fos.close();
    }

```

[参考链接](https://stackoverflow.com/questions/33696488/getting-bitmap-from-vector-drawable)

