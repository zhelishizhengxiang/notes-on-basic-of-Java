## 1）FileReader
该类的类继承图如下图所示：
![](assets/05FileReader/file-20250327220538148.png)

相关构造器以及方法

![](assets/05FileReader/file-20250327220606869.png)
* 注意：字符流对于中英文都有的文件来说不会出现乱码，因为是一个字符一个字符读的，一个汉字为一个字符；而使用字节流读的话中文会出现乱码，因为UTF-8字符集的一个汉字是三个字节，他是一个字节一个字节读肯定会产生乱码

使用实例：

![](assets/05FileReader/file-20250327221536513.png)
![](assets/05FileReader/file-20250327221735012.png)


