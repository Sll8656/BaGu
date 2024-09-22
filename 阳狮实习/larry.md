启动的时候报错缺少两个sqlSessionFactory，解决方案：在issus中看到的，加入连接池依赖即可。

希望看到打印的sql语句，在mybatisFlex官方文档找sql打印

希望报异常会返回给前端，使用全局异常处理器。

>（1）@RestControllerAdvice是一个组合注解，由@ControllerAdvice、@ResponseBody组成，而@ControllerAdvice继承了
>@Component，因此@RestControllerAdvice本质上是个Component，用于定义@ExceptionHandler，@InitBinder和@ModelAttribute方法，适用于所有使用@RequestMapping方法。
>（2）@RestControllerAdvice的特点：
>1.通过@ControllerAdvice注解可以将对于控制器的全局配置放在同一个位置。
>2.注解了@RestControllerAdvice的类的方法可以使用@ExceptionHandler、@InitBinder、@ModelAttribute注解到方法上。
>3.@RestControllerAdvice注解将作用在所有注解了@RequestMapping的控制器的方法上。
>4.@ExceptionHandler：用于指定异常处理方法。当与@RestControllerAdvice配合使用时，用于全局处理控制器里的异常。
>5.@InitBinder：用来设置WebDataBinder，用于自动绑定前台请求参数到Model中。
>6.@ModelAttribute：本来作用是绑定键值对到Model中，当与@ControllerAdvice配合使用时，可以让全局的@RequestMapping都能获得在此处设置的键值对



![image-20240922092737293](https://s2.loli.net/2024/09/22/RxqHjv8pdPMnDmT.png)

---

```java
String storedToken = (String) redisTemplate.opsForValue().get(userId);
```

明明redis中有，为什么获取不到？

答：写成了RedisTemplate，应该用StringRedisTemplate，他们两个的序列化机制是不一样的。

---

  String userId = ""; 为什么它提示我这个要初始化为blank，不初始化会出现什么问题

---

全局异常处理器中：

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ErrorResponse> handleException(Exception ex, WebRequest request) {
    ErrorResponse errorResponse = new ErrorResponse();
    errorResponse.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());
    errorResponse.setMessage(ex.getMessage());
    return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
}
```

这个一定要用到ex.getMessage()，他会把异常记录。

---

为什么maven编译能通过，但是点击重新加载maven项目，就出错了