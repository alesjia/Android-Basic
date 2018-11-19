# LruCache源码分析
## 文档说明
首先我们先看看这个类的文档说明，大致包括以下五个方面。
1. LruCache缓存的数据持有强引用，所以在不需要的时候需要清除缓存
2. 如果LruCache持有的数据清除时需要显式释放，那么请重写entryRemoved方法
3. LruCache的数据清除策略是最少使用策略，如其名（Lru Least Recently Used）
4. LruCache是线程安全的，这意味着你可以在多线程中直接使用，不需要使用Syncronize等关键字来保证数据一致性以及可见性
5. LruCache底层是使用的是LinkedHashMap来保存数据的，但不允许key和value是null

那么这五个方面到底想告诉我们什么，或者我们该如何做？下面将对每个方面都做详细的说明
一、缓存持有强引用，即使这些内存你不用了，这些被占用的内存是不会自动释放的。所以我们在使用LruCache的时候，如果不需要再用LruCache时，需要手动释放。也就是需要调用evictAll()方法来释放缓存。
二、什么叫显式释放？对于一般的对象，我们一般都不需要进行显式释放。内存管理工作由JVM的垃圾回收机制来自动处理，不需要人为参与。但是对于有些对象，则需要显式释放，如Bitmap需要调用recycle()方法来释放内存。
三、LruCache的最近最少使用缓存清除策略是如何实现的呢？这个主要通过在创建LinkedHashMap对象的时候，设置accessOrder为true，这样就可以实现在数据插入、获取时，最近插入或者获取的数据项一直排在链表头。相对应的如果一个数据项好久没有被使用过，那么这个数据项会沉到链表尾。那么如果此时又有一个新的数据插入到该链表时，而数据的总大小超过了设置的最大缓存，则需要从链表尾删除一个或者多个最近很少使用的数据，给新插入的数据挪出空间以保证不超过最大缓存。
四、LruCache的方法或者代码块通过synchronized的关键字保证线程安全
五、在put和get方法的时候对key和value做了null判断，如果是null，直接抛出NullPointerException异常

## 代码分析
### 1.成员变量
```
private final LinkedHashMap<K, V> map;

*/** Size of this cache in units. Not necessarily the number of elements. */*
private int size;//目前占用的缓存大小
private int maxSize;//设置的最大缓存，在实际存储时不会超过这个大小。如果超过，则会将最近很少使用的数据项清除掉，从而保证不超过这个最大值。

//主要用作统计
private int putCount;//调用put存入数据的次数
private int createCount;//调用create创建数据项的次数，默认情况下是0，除非你重写了create方法
private int evictionCount;//由于空间超过最大缓存而导致的数据清除次数
private int hitCount;//就是缓存命中数，也就是通过get拿到数据的次数
private int missCount;//缓存未命中数
```

### 2.构造函数
```
public LruCache(int maxSize) {
    if (maxSize <= 0) {
        throw new IllegalArgumentException("maxSize <= 0");
    }
    this.maxSize = maxSize;//设置最大缓存
    this.map = new LinkedHashMap<K, V>(0, 0.75f, true);//这个true非常值得注意，通过这个就可以保证最近put或者get的数据放在链表的最前面
}

```

### 3.V get(K key)函数
```
public final V get(K key) {
    if (key == null) {//不允许key值为null
        throw new NullPointerException("key == null");
    }

    V mapValue;
    synchronized (this) {
        mapValue = map.get(key);
        if (mapValue != null) {
            hitCount++;//如果从缓存拿到数据，更新命中数
            return mapValue;//直接拿到缓存数据
        }
        missCount++;//如果没有拿到缓存，更新失败数
    }

    /*
     * Attempt to create a value. This may take a long time, and the map
     * may be different when create() returns. If a conflicting value was
     * added to the map while create() was working, we leave that value in
     * the map and release the created value.
     */

    V createdValue = create(key);
    if (createdValue == null) {
        return null;//如果没有重写create，那么直接返回
    }

    synchronized (this) {
        createCount++;//如果createdValue不为空，更新创建数据数
        mapValue = map.put(key, createdValue);//将创建的数据存入缓存

        if (mapValue != null) {
            // There was a conflict so undo that last put
            map.put(key, mapValue);//应该不会存在这个情况？
        } else {
            size += safeSizeOf(key, createdValue);//更新缓存大小
        }
    }

    if (mapValue != null) {
        entryRemoved(false, key, createdValue, mapValue);
        return mapValue;
    } else {
        trimToSize(maxSize);//保证不超过最大缓存
        return createdValue;//将创建的缓存数据项返回
    }
}

```

### 4.V put(K key, V value)
```
public final V put(K key, V value) {
    if (key == null || value == null) {
        throw new NullPointerException("key == null || value == null");
    }//不允许key，value值为空

    V previous;
    synchronized (this) {
        putCount++;//更新添加数
        size += safeSizeOf(key, value);//更新缓存大小
        previous = map.put(key, value);//将缓存放入linkedHashmap
        if (previous != null) {
            size -= safeSizeOf(key, previous);//更新缓存大小
        }
    }

    if (previous != null) {
        entryRemoved(false, key, previous, value);//释放旧数据
    }

    trimToSize(maxSize);//保证缓存不超过最大缓存
    return previous;//返回旧的数据
}

```

## 4.trimToSize(int maxSize)
```
public void trimToSize(int maxSize) {
    while (true) {
        K key;
        V value;
        synchronized (this) {
            if (size < 0 || (map.isEmpty() && size != 0)) {
                throw new IllegalStateException(getClass().getName()
                        + ".sizeOf() is reporting inconsistent results!");
            }

            if (size <= maxSize) {//如果是目前的缓存没有超过想要限制的缓存大小，直接返回
                break;
            }

            Map.Entry<K, V> toEvict = map.eldest();//从链表最后删除最近很少使用的数据项
            if (toEvict == null) {
                break;
            }

            key = toEvict.getKey();
            value = toEvict.getValue();
            map.remove(key);
            size -= safeSizeOf(key, value);//更新缓存大小
            evictionCount++;//更新删除的数据数
        }

        entryRemoved(true, key, value, null);//将这个数据项释放
    }
}

```

### 5.V remove(K key)
```
public final V remove(K key) {
    if (key == null) {
        throw new NullPointerException("key == null");
    }

    V previous;
    synchronized (this) {
        previous = map.remove(key);//删除数据
        if (previous != null) {
            size -= safeSizeOf(key, previous);//更新缓存大小
        }
    }

    if (previous != null) {
        entryRemoved(false, key, previous, null);//将这个数据项释放
    }

    return previous;
}

```
### 6.int sizeOf(K key, V value)
```
protected int sizeOf(K key, V value) {
    return 1;//默认实现返回1，这是因为默认实现maxSize代表的是缓存数据项的数目。当增加一个数据项，那么size会增加1，而删除一个数据项，则size减1.
不过一般情况下不会用LruCache来缓存普通的数据项，如javaBean等。目前主要使用LruCache来缓存图片。
}

可以看看Picasso的cache实现
@Override protected int sizeOf(String key, BitmapAndSize value) {
  return value.byteCount;//返回一个图片的大小
}
```
