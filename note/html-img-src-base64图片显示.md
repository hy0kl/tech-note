#  html img src base64 图片显示

[From](http://blog.csdn.net/samqingqing/article/details/7532141)

大家可能注意到了，网页上有些图片的src或css背景图片的url后面跟了一大串字符，比如：

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAAkCAYAAABIdFAMAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAHhJREFUeNo8zjsOxCAMBFB/KEAUFFR0Cbng3nQPw68ArZdAlOZppPFIBhH5EAB8b+Tlt9MYQ6i1BuqFaq1CKSVcxZ2Acs6406KUgpt5/LCKuVgz5BDCSb13ZO99ZOdcZGvt4mJjzMVKqcha68iIePB86GAiOv8CDADlIUQBs7MD3wAAAABJRU5ErkJggg%3D%3D
```

那么这是什么呢？这是Data URI scheme。

Data URI scheme是在RFC2397中定义的，目的是将一些小的数据，直接嵌入到网页中，从而不用再从外部文件载入。比如上面那串字符，其实是一张小图片，将这些字符复制黏贴到火狐的地址栏中并转到，就能看到它了，一张1X36的白灰png图片。

在上面的Data URI中，data表示取得数据的协定名称，image/png 是数据类型名称，base64 是数据的编码方法，逗号后面就是这个image/png文件base64编码后的数据。

目前，Data URI scheme支持的类型有：

```
data:text/plain,文本数据
data:text/html,HTML代码
data:text/html;base64,base64编码的HTML代码
data:text/css,CSS代码
data:text/css;base64,base64编码的CSS代码
data:text/javascript,Javascript代码
data:text/javascript;base64,base64编码的Javascript代码
data:image/gif;base64,base64编码的gif图片数据
data:image/png;base64,base64编码的png图片数据
data:image/jpeg;base64,base64编码的jpeg图片数据
data:image/x-icon;base64,base64编码的icon图片数据
```

base64简单地说，它把一些 8-bit 数据翻译成标准 ASCII 字符，网上有很多免费的base64 编码和解码的工具，在PHP中可以用函数base64_encode() 进行编码，如echo base64_encode(file_get_contents(‘wg.png’));

目前，IE8、Firfox、Chrome、Opera浏览器都支持这种小文件嵌入。

举个图片的例子：

```
网页中一张图片可以这样显示：
<img src=“http://www.letuknowit.com/images/wg.png”/>
也可以这样显示：
<img src=“data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAAkCAYAAABIdFAMAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAHhJREFUeNo8zjsOxCAMBFB/KEAUFFR0Cbng3nQPw68ArZdAlOZppPFIBhH5EAB8b+Tlt9MYQ6i1BuqFaq1CKSVcxZ2Acs6406KUgpt5/LCKuVgz5BDCSb13ZO99ZOdcZGvt4mJjzMVKqcha68iIePB86GAiOv8CDADlIUQBs7MD3wAAAABJRU5ErkJggg%3D%3D” />
```

我们把图像文件的内容直接写在了HTML 文件中，这样做的好处是，节省了一个HTTP 请求。坏处呢，就是浏览器不会缓存这种图像。大家可以根据实际情况进行自由取舍。
