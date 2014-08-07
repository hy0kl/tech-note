# php 开发接口时空数组和空对象

现象描述:
和后端约定 ext 字段是对象,但初始化为 array(),如果后端有没有结果,最终给 app 输出 json 时 ext 的值变成了 [],即变成数组了.

解决方法:
将 ext 初始化对象 new ArrayObject().

