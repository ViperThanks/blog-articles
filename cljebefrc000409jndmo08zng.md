---
title: "基于Javaの算法板子"
datePublished: Thu May 11 2023 13:17:04 GMT+0000 (Coordinated Universal Time)
cuid: cljebefrc000409jndmo08zng
slug: 5lin5piv
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687871680280/14af24e7-cead-46d6-859f-25f52feb0642.jpeg
tags: algorithms, java

---

$$基于yxc的模板$$

# 数组排序

## 快排

随机挑选一个基准元素（一般算法竞赛取中间就行了）并且偏移左右指针单位一, 遍历数组 , 一次遍历后,左边元素比基准元素小 ,右边反之, 递归这个步骤直到排序完成

```java
  public static void quick_sort(int[] arr, int left, int right) {
    if (left >= right) return;

    int x = arr[left + right >> 1], i = left - 1, j = right + 1;

    while (i < j) {
      do i++; while (arr[i] < x);
      do j--; while (arr[j] > x);
      if (i < j)
        AlgoUtils.swap(arr, i, j);
    }

    quick_sort(arr, left, j);
    quick_sort(arr, j + 1, right);
  }
```

## merge 排

递归排序，先递归到最深，`merge_sort(arr, l, mid)` 然后双指针扫上下数组，tmp排好序，再扫尾，最后把 tmp 数组换回 arr 数组

```java
  public static void merge_sort(int[] arr, int l, int r) {
    if (l >= r) return;
    int mid = l + r >> 1;

    merge_sort(arr, l, mid);
    merge_sort(arr, mid + 1, r);

    int[] tmp = new int[r - l + 1];
    int k = 0, i = l, j = mid + 1;

    while (i <= mid && j <= r) {
      if (arr[i] <= arr[j])
        tmp[k++] = arr[i++];
      else
        tmp[k++] = arr[j++];
    }
    //扫尾
    while (i <= mid)
      tmp[k++] = arr[i++];
    while (j <= r)
      tmp[k++] = arr[j++];
    for (i = l, j = 0; i <= r; )
      arr[i++] = tmp[j++];
  }
```

基于 [AcWing](https://www.acwing.com/) 网址学习算法的总结的模板