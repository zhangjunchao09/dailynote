getResourceAsStream和getResource的用法及Demo实例

     用JAVA获取文件，听似简单，但对于很多像我这样的新人来说，还是掌握颇浅，用起来感觉颇深，大家最经常用的，就是用JAVA的File类，如要取得 D:/test.txt文件，就会这样用File file = newFile("D:/test.txt");这样用有什么问题，相信大家都知道，就是路径硬编码，对于JAVA精神来说，应用应该一次成型，到处可 用，并且从现实应用来讲，最终生成的应用也会部署到Windows外的操作系统中，对于linux来说，在应用中用了c:/这样的字样，就是失败，所以， 我们应该尽量避免使用硬编码，即直接使用绝对路径。

  　在Servlet应用中，有一个getRealPath(String str)的方法，这个方法尽管也可以动态地获得文件的路径，不秘直接手写绝对路径，但这也是一个不被建议使用的方法，那么，我们有什么方法可以更好地获得文件呢?       那就是Class.getResource()与Class.getResourceAsStream()方法！    

一、getResourceAsStream用法 首先，Java中的getResourceAsStream有以下几种： 

1. Class.getResourceAsStream(String path) ：  path 不以’/'开头时默认是从此类所在的包下取资源，以’/'开头则是从 ClassPath根下获取。其只是通过path构造一个绝对路径，最终还是由ClassLoader获取资源。      2. Class.getClassLoader.getResourceAsStream(String path) ： 默认则是从ClassPath根下获取，path不能以’/'开头，最终是由   ClassLoader获取资源。

3. ServletContext. getResourceAsStream(String path)： 默认从WebAPP根目录下取资源，Tomcat下path是否以’/'开头无所谓，   当然这和具体的容器实现有关。    

4. Jsp下的application内置对象就是上面的ServletContext的一种实现。    其次，getResourceAsStream 用法大致有以下几种：

第一： 要加载的文件和.class文件在同一目录下，例如：com.x.y 下有类me.class ,同时有资源文件myfile.xml 那么，应该有如下代码：    Java代码   me.class.getResourceAsStream("myfile.xml");       

第二：在me.class目录的子目录下，例如：com.x.y 下有类me.class ,同时在 com.x.y.file 目录下有资源文件myfile.xml   那么，应该有如下代码：    Java代码   me.class.getResourceAsStream("file/myfile.xml");        

第三：不在me.class目录下，也不在子目录下，例如：com.x.y 下有类me.class ,同时在 com.x.file 目录下有资源文件myfile.xml   那么，应该有如下代码：     Java代码   me.class.getResourceAsStream("/com/x/file/myfile.xml"); 