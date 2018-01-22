# class和static

## 概念
> static和class，在Swift中表示 “类型范围作用域” 这一概念

## class和static区别
* 在非class的类型上下文中：我们统一使用static来描述类型作用域。

* class类型的上下文中：class和static均可使用，当然他们此时有不同语义：class修饰计算属性和类方法，static修饰静态方法和静态属性（静态的在编译期决定）。

## 补充
* class不能修饰非计算属性（因为在Objective-C中就没有类变量这个概念，为了运行时的统一和兼容，暂时不太方便添加这个特性）