---
title: 一个整型值益出的错误
date: 2017-02-21 11:37:54
tags: Java
---

今天遇到一个整型益处的错误，我觉得值得反思下，具体是这样的：

```
obj = {
   int priority();		//业务顺序
}
```

有个乱序的arr = [obj\_C, obj\_B, obj\_C, obj\_A, obj\_F, obj\_D]，需要排序，于是默认使用了Comparator接口，通过他的compare(T lhs, T rhs)方法返回一个整型，整型的大小决定左右参数的排序，这样写

```
int compare(Obj lhs, Obj rhs) {
	return lhs.proprity() - rhs.proprity();
}
```

然后需求决定数组里面obj_A永远排在首位，于是为了简便我把 obj_A的类实现成这样：

```
obj_A = {
    int priority() {
        return Integer.MIN_VALUE;
    }
}
```

看似没啥问题，``` obj_A.proprity() - obj_other().proprity ```  值很小 或者 ``` obj_other().proprity － obj_A.proprity() ``` 值很大，排完序应该在第一位，可是事实却是永远排在最后一位。

事实上我遇到了整型益出的问题，Integer.MIN_VALUE - obj_other.proprity()，这样的计算会发生益出，得到的值很大, 并不是预期的值，同理也一样。

于是问题迎刃而解，只要给obj_A返回一个适当大小的负数即可，保证不会溢出。



> 记录下：负数在电脑中是怎么相加的？

答：
3的原码：0000 0011

3的反码：1111 1100

3的补码：    （即反码加1）
1111 1101

可以看出3的源码与反码相加为1111 1111为﹣127而与补码相加刚好为0000 0000为0这时补码就可以作为源码的负数表示正常的参与运算，在计算机中一般用加法代替减法，把要减的数换成补码再相加。负数加法也是一样的。
