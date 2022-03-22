ArrayList常量：
``` java
private static final long serialVersionUID = 8683452581122892189L;  
//用于空实例的共享空数组实例  
Object[] EMPTY_ELEMENTDATA = {};  
//用于默认大小的空实例的共享空数组实例。我们将其与 EMPTY_ELEMENTDATA 区分开来，以了解添加第一个元素时要膨胀多少。
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};  
//存储 ArrayList 元素的数组缓冲区。 ArrayList 的容量就是这个数组缓冲区的长度。当添加第一个元素时，任何具有 elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA 的空 ArrayList 都将扩展为 DEFAULT_CAPACITY。  
transient Object[] elementData; 
//ArrayList的容量  
private int size;
//默认初始化大小
private static final int DEFAULT_CAPACITY = 10;
```
构造函数：
构造函数有三个
一个是指定初始化的大小的
一个是没指定大小的
还有一个是按照集合的迭代器返回的顺序构造一个包含指定集合元素的列表
```java 
public ArrayList(int initialCapaicty) {  
    if (initialCapacity > 0) {  
        this.elementData = new Object[initialCapacity];  
    } else if (initialCapacity == 0) {  
        this.elementData = EMPTY_ELEMENTDATA;  
    } else {  
        throw new IllegalArgumentException("Illegal Capacity: "+  
                                           initialCapacity);  
    }  
}
//不指定初始容量，创建一个空实例
public ArrayList() {  
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
}

public ArrayList(Collection<? extends E> c) {  
    Object[] a = c.toArray();  
    if ((size = a.length) != 0) {  
        if (c.getClass() == ArrayList.class) {  
            elementData = a;  
        } else {  
            elementData = Arrays.copyOf(a, size, Object[].class);  
        }  
    } else {  
        // replace with empty array.  
 elementData = EMPTY_ELEMENTDATA;  
    }  
}
```

