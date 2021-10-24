---
title: ArrayList源码
categories:
  - JDK
index_img: >-
  https://199794.oss-cn-shanghai.aliyuncs.com/blog//2019-04-22%20135447_gaitubao_1600x900_1604366529483.jpg
date: 2021-10-24 15:01:31
---


```
public class MyArrayList<E> implements Iterable<E> {

    private static final int DEFAULT_CAPACITY = 10;  //初始容量

    transient Object[] elementData;

    //空数组实例
    private static final Object[] EMPTY_ELEMENTDATA = {};

    //默认空数组实例,以了解添加第一个元素时,空间需要扩大多少
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    //与快速失败机制有关的属性,记录容器大小是否更改
    protected transient int modCount = 0;

    //虚拟机规定的最大数组大小,超过可能会引发OutOfMemoryError
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    //数组大小
    private int size;

    public MyArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }


    public MyArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("容量错误");
        }
    }

    public void add(int index, E element) {
        //todo
    }

    public boolean add(E e) {
        //保证容量
        ensureCapacityInternal(size + 1);
        elementData[size++] = e;
        return true;

    }

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;

    }


    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0) {
            grow(minCapacity);
        }
    }

    //扩容核心方法
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        //这边右移一位表示一半,所以扩容为原来的1.5大小
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        if (newCapacity - MAX_ARRAY_SIZE > 0) {
            newCapacity = hugeCapacity(minCapacity);
        }
        //复制扩容数组
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++) {
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
            }
        } else {
            for (int index = 0; index < size; index++) {
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
            }
        }

        return false;
    }

    public void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, numMoved);
        }
        elementData[--size] = null;
    }


    private static int hugeCapacity(int minCapaCity) {
        if (minCapaCity < 0) {
            throw new OutOfMemoryError();
        }
        return minCapaCity > MAX_ARRAY_SIZE ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
    }

    //返回迭代器
    @Override
    public Iterator<E> iterator() {
        return null;
    }

    //内部迭代器
    private class Itr implements Iterator<E> {
        int curosr;  //下一个元素index
        int lastRet = -1;  //  最后一个元素的index;

        int expectedModCount = modCount;  //快速失败机制

        Itr() {
        }

        @Override
        public boolean hasNext() {
            return curosr != size;
        }

        @Override
        public E next() {
            checkForComodification();
            int i = curosr;
            if (i >= size) {
                throw new NoSuchElementException();
            }

            Object[] elementData = MyArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            curosr = i + 1;
            return (E) elementData[lastRet = i];

        }

        @Override
        public void remove() {
            if (lastRet < 0) {
                throw new IllegalStateException();
            }
            checkForComodification();
            MyArrayList.this.remove(lastRet);
            curosr = lastRet;
            lastRet = -1;
            expectedModCount = modCount;

        }

        private void checkForComodification() {
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
        }
    }
}
```