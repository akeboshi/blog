---
title: JavaのArrayListの配列の大きさについて
date: 2021-03-06T12:29:18.420Z
draft: false
---


Java のArrayListはListの実装でみなさんよく使ってると思います。

配列を使ってListを実装しているが、どうやって配列の大きさを決めて配列を作っているかを見ていく


- ref: 
  - ArrayList: https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/ArrayList.java
  - ArraysSupport: https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/jdk/internal/util/ArraysSupport.java



## 結論


- new するときは想像の通りで、何も指定されなければ空、指定されればそのサイズの配列が内部で作られる。
- add するときは、(addされる前の配列の大きさ >> 1) 分増えるので、1/2になる
- addAll するときは、addするときと同じでサイズで配列のサイズが足りれば、addするときと同じ。足りなければ、必要な分丁度の配列が作られる。
- 要素を消しても初期サイズに戻ったりはしない


## 実装を読む


### new


`new ArrayList()` しただけだと、空の配列が作られる



```java
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA; // 空の配列
}
```



`new ArrayList(initialCapacity)` すると、指定したサイズの大きさの配列が作られる。




```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                            initialCapacity);
    }
}
```


### add


new ArrayList(5) をして、中身が [1,2,3,4,5] のを例として


- `add(6)` をした流れを見てみる

  - 内部配列 (elementData) の大きさ: 5
  - size: 5


```java
public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}
```



private なaddを見る


```java
// e: 6
// elementData.length: 5
// size: 5
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow(); // ここで、配列を大きくしている
    elementData[s] = e;
    size = s + 1;
}
```


```java
// size: 5
private Object[] grow() {
    return grow(size + 1);
}

// minCapacity: 6
private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length; // oldCapacity: 5
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity,
                oldCapacity >> 1           ); // ArraysSupport.newLength(5, 1, 5 >> 1) で呼び出し
        return elementData = Arrays.copyOf(elementData, newCapacity); // 配列の作り直し
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
```


newCapacity の大きさで配列がコピーされる確保されるので、newCapacityの呼び出しを見ていく


```java
// oldLength: 5
// minGrowth: 1
// prefGrowth: 5 >> 1 = 2
public static int newLength(int oldLength, int minGrowth, int prefGrowth) {
    int newLength = Math.max(minGrowth, prefGrowth) + oldLength; // Math.max(1, 5 >> 1) + 5 = 7
    if (newLength - MAX_ARRAY_LENGTH <= 0) {
        return newLength; // 7 が返る
    }
    return hugeLength(oldLength, minGrowth);
}
```


という形で、`Arrays.copyOf(elementData, 7)` が実行され、内部配列 7 の大きさで作り直される


---


実際に試してみる


```java
try {
    List<Integer> list = new ArrayList<>(5);
    for (Integer i = 1; i <= 5 ; i++) {
        list.add(i);
    }

    Field f = list.getClass().getDeclaredField("elementData");
    f.setAccessible(true);  

    System.out.println("add前のelementData.length: " + ((Object[])f.get(list)).length);
    System.out.println("add前のsize: " + list.size());

    list.add(6);

    System.out.println("add後のelementData.length: " + ((Object[])f.get(list)).length);
    System.out.println("add後のsize: " + list.size());
} catch (Exception e) {
    e.printStackTrace();
}
```


```
add前のelementData.length: 5
add前のsize: 5
add後のelementData.length: 7
add後のsize: 6
```


ということで、1/2倍増えていることがわかる


### addAll


add で grow 関数を読んでいると思うので、詳しくは割愛する。

new ArrayList(5) をして、中身が [1,2,3,4,5] のを例として ArrayList(中身が[6,7,8,9,10])される場合新しい配列のサイズは以下の流れで10のcapacityの配列が内部で生成される (Javaとして正しくない表記をしています)

0. addAll([6,7,8,9,10])
1. grow(5 + 5)
2. ArraysSupport.newLength(5, 5, 5 >> 1)
3. Math.max(5, 5 >> 1) + 5 の結果 10 が newCapacity として返る





```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    modCount++;
    int numNew = a.length;
    if (numNew == 0)
        return false;
    Object[] elementData;
    final int s;
    if (numNew > (elementData = this.elementData).length - (s = size))
        elementData = grow(s + numNew);
    System.arraycopy(a, 0, elementData, s, numNew);
    size = s + numNew;
    return true;
}
```

---


実際に動かしてみる


```java
try{
    List<Integer> list = new ArrayList<>(5);
    for (Integer i = 1; i <= 5 ; i++) {
        list.add(i);
    }

    Field f = list.getClass().getDeclaredField("elementData");
    f.setAccessible(true);  

    System.out.println("add前のelementData.length: " + ((Object[])f.get(list)).length);
    System.out.println("add前のsize: " + list.size());

    List<Integer> list2 = new ArrayList<>();
    for (Integer i = 6; i <= 10 ; i++) {
        list2.add(i);
    }

    list.addAll(list2);

    System.out.println("add後のelementData.length: " + ((Object[])f.get(list)).length);
    System.out.println("add後のsize: " + list.size());
} catch (Exception e) {
    e.printStackTrace();
}
```


```
add前のelementData.length: 5
add前のsize: 5
add後のelementData.length: 10
add後のsize: 10
```


内部配列の大きさが10になっていることがわかる