# Category和Extension
Category（类别）和Extension（类扩展），虽然有人说extension是一个特殊的category，也有人将extension叫做匿名分类，但是其实两者差别很大。

## Extension
* 只能在类定义文件中出现，编译期与类定义一同编译，实质上是类定义的一部分。
* 属性、方法默认都是private。

## Category
* 运行时与类定义文件组合形成类。
* 因为是在运行时整合的，所以不能添加属性。