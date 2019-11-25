---
title: 面试之---算法相关
categories: interview
tags: 
  - 总结
  - 算法
date: 2018-09-16 21:01:51
description: 包含算法相关面试题汇总
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](interview-algorithm/interview.jpg)

> 持续维护java面试题系列博文，发现好的面试题会更新进来。总结的并不只是如何回答面试官，而是这道题涉及到的知识点，明白了相关知识点和面试官要问的点，回答起来就不是问题了。

## 排序算法

### 用Java写一个冒泡排序。

冒泡排序几乎是个程序员都写得出来，但是面试的时候如何写一个逼格高的冒泡排序却不是每个人都能做到，下面提供一个参考代码：

```java
import java.util.Comparator;

/**
 * 排序器接口(策略模式: 将算法封装到具有共同接口的独立的类中使得它们可以相互替换)
 */
public interface Sorter {

   /**
    * 排序
    * @param list 待排序的数组
    */
   public <T extends Comparable<T>> void sort(T[] list);

   /**
    * 排序
    * @param list 待排序的数组
    * @param comp 比较两个对象的比较器
    */
   public <T> void sort(T[] list, Comparator<T> comp);
}
```

```java
import java.util.Comparator;

/**
 * 冒泡排序
 */
public class BubbleSorter implements Sorter {

    @Override
    public <T extends Comparable<T>> void sort(T[] list) {
        boolean swapped = true;
        for (int i = 1, len = list.length; i < len && swapped; ++i) {
            swapped = false;
            for (int j = 0; j < len - i; ++j) {
                if (list[j].compareTo(list[j + 1]) > 0) {
                    T temp = list[j];
                    list[j] = list[j + 1];
                    list[j + 1] = temp;
                    swapped = true;
                }
            }
        }
    }

    @Override
    public <T> void sort(T[] list, Comparator<T> comp) {
        boolean swapped = true;
        for (int i = 1, len = list.length; i < len && swapped; ++i) {
            swapped = false;
            for (int j = 0; j < len - i; ++j) {
                if (comp.compare(list[j], list[j + 1]) > 0) {
                    T temp = list[j];
                    list[j] = list[j + 1];
                    list[j + 1] = temp;
                    swapped = true;
                }
            }
        }
    }
}
```



## 查找算法

### 用Java写一个折半查找。

折半查找，也称二分查找、二分搜索，是一种在有序数组中查找某一特定元素的搜索算法。搜素过程从数组的中间元素开始，如果中间元素正好是要查找的元素，则搜素过程结束；如果某一特定元素大于或者小于中间元素，则在数组大于或小于中间元素的那一半中查找，而且跟开始一样从中间元素开始比较。如果在某一步骤数组已经为空，则表示找不到指定的元素。这种搜索算法每一次比较都使搜索范围缩小一半，其时间复杂度是O(logN)。

```java
import java.util.Comparator;

public class MyUtil {

   public static <T extends Comparable<T>> int binarySearch(T[] x, T key) {
      return binarySearch(x, 0, x.length- 1, key);
   }

   // 使用循环实现的二分查找
   public static <T> int binarySearch(T[] x, T key, Comparator<T> comp) {
      int low = 0;
      int high = x.length - 1;
      while (low <= high) {
          int mid = (low + high) >>> 1;
          int cmp = comp.compare(x[mid], key);
          if (cmp < 0) {
            low= mid + 1;
          }
          else if (cmp > 0) {
            high= mid - 1;
          }
          else {
            return mid;
          }
      }
      return -1;
   }

   // 使用递归实现的二分查找
   private static<T extends Comparable<T>> int binarySearch(T[] x, int low, int high, T key) {
      if(low <= high) {
        int mid = low + ((high -low) >> 1);
        if(key.compareTo(x[mid])== 0) {
           return mid;
        }
        else if(key.compareTo(x[mid])< 0) {
           return binarySearch(x,low, mid - 1, key);
        }
        else {
           return binarySearch(x,mid + 1, high, key);
        }
      }
      return -1;
   }
}
```

> 说明：上面的代码中给出了折半查找的两个版本，一个用递归实现，一个用循环实现。需要注意的是计算中间位置时不应该使用(high+ low) / 2的方式，因为加法运算可能导致整数越界，这里应该使用以下三种方式之一：low + (high – low) / 2或low + (high – low) >> 1或(low + high) >>> 1（>>>是逻辑右移，是不带符号位的右移）









> 参考链接：
>
> https://blog.csdn.net/jackfrued/article/details/44921941