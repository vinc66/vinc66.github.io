---
title: java_clone
date: 2017-01-18 11:44:10
tags:
- java
---

---

#### 1.创建新对象用new和clone的不同与相同
            new的本意是分配内存，是要知道了操作符后面的类型才能分配。分配完内存，调用构造函数，填充对象的各个域，
        然后初始化，初始化方法完成后，创建完毕，将对象引用的地址暴露给外部调用。clone是new分配的内存和源对象相同，
        使用源对象的域创建新的对象，然后创建新的对象引用地址暴露给外部。
#### 2.clone的两种方式:深拷贝,浅拷贝
            浅拷贝就是源对象和新创建的对象引用地址不同，但是内部成员变量的引用地址和内容是相同的。默认的clone方
        法是浅拷贝。如果想要进行深拷贝需要实现Cloneable 接口，重写clone方法。
#### 3.举个例子
```
    public class Man implements Cloneable{
        public Hair hair;
        Man(){
            hair = new Hair();
        }
        @Override
        protected Object clone() throws CloneNotSupportedException {
            Man clone = (Man) super.clone();
            clone.hair = (Hair) hair.clone();
            return clone;
        }
        public static void main(String[] args) throws CloneNotSupportedException {
            Man man = new Man();
            Hair hair = man.hair;
            Man clone = (Man) man.clone();
            Hair hairclone = clone.hair;
    //        man != clone ;
    //        hair != hairclone;
        }


        class Hair implements Cloneable{
            String color;
            @Override
            protected Object clone() throws CloneNotSupportedException {
                return super.clone();
            }
        }
    }
```


---
