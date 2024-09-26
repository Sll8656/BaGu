# 异常

**异常处理（什么时候try catch什么时候抛出）**

关键点分析：
 **1、throw 异常之后，后面的代码将不会执行
 2、try-catch时，即使catch到了异常之后，后面的代码还是会继续执行**

getConnection时没有必要采用trycatch 因为如果连数据库都连接不上，就不用谈后面对数据库的操作了。

释放资源的时候trycatch是必要的，因为不止关了一个资源。
如本例中，如果在处理statement的异常时，抛出了，后面的conn资源就关闭不了了，所以采用try-catch是恰当的。