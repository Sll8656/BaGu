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

为什么maven编译能通过，但是点击重新加载maven项目，就出错了:

解决：把阿里云镜像打开

---

我http://47.100.251.64:15672/，连接自己的服务器，为什么浏览器直接说链接不安全

用自己的网就行了，公司网不行。

---

错误："JSON parse error: Cannot deserialize value of type `java.lang.Long` from Array value (token `JsonToken.START_ARRAY`); nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot deserialize value of type `java.lang.Long` from Array value (token `JsonToken.START_ARRAY`)\n at [Source: (org.spring

答：传参错误，我传id为1，而不是数组[1]。

---

**问：**

为什么：UpdateChain.*of*(Article.class)
        .eq(Article::getId, id)
        .set(Article::getIsPublished, *IS_PUBLISHED*)
        .update();
的sql语句是：UPDATE `article` SET `is_published` = ? WHERE `deleted` = ?，并没有更新指定的id
**答：**

Controller中没有正确地传入参数，导致根本没拿到id

---

用户的分页查询，写在`AdminController`中

博客的分页查询，写在`ArticleController`中

因为：只有管理员才能对用户进行分页查询，

---

```java
String token = JwtTokenUtils.getToken(admin.getId().toString(), admin.getSalt());
redisTemplate.opsForValue().set(ADMIN_KEY + admin.getId().toString(), token, TOKEN_EXPIRATION, TimeUnit.SECONDS);      
```

问：观察redis，发现redis中的token和我的token不一致，是什么原因？

答：我只看了当前的最后三位，实际上进度条应该往后拉到底看最后3位。

---

使用Id作为redis的key时，因为用户和管理员采用不同的表，所以id会重复。为了解决这个问题，需要在redis中加上前缀，即user:userId, Admin:admin。区分用户和管理员。但是在拦截器中只有**先区分好**是用户还是管理员，才能根据角色去存放进redis。

先区分好用户还是管理员的方式：

1. 通过uri，用户相关接口包含/user，就是用户：后来发现不行，因为管理员也会调用user的接口
2. 通过token，在登录的时候生成token，生成token的时候，把角色放进token的claim ("role" , Admin)

最终采用方式2，解码：`String userType = JWT.*decode*(token).getClaim("role").asString();`

---

article分页查询为什么查到的数据不全？

答：我用的是in，用in的话第三个参数的条件直接作为占位符参数给传进来了。应该用like模糊查询

```java
 wrapper.like(Article::getTitle, articleQuery.getTitle(), StrUtil.isNotBlank(articleQuery.getTitle()))
        .like(Article::getContent, articleQuery.getContent(), StrUtil.isNotBlank(articleQuery.getContent()))
        .orderBy("id", true);
```

构造分页查询时，通过列表下拉值来选择构建很多**查询块**的

![image-20240925115646418](https://s2.loli.net/2024/09/25/Ib9F8jHpKAePCsy.png)

使用`List<String>`构造查询条件，然后wrapper中使用in，条件判空用`CollUtil.isNotEmpty`



prd-All没有条件查询

production-In是分块查询

prd-before是input查询

```java
package com.publicisgroupe.procure.mvp.procure.config.aspect;

import cn.hutool.core.util.StrUtil;
import com.publicisgroupe.procure.mvp.common.core.annotation.OperationLog;
import com.publicisgroupe.procure.mvp.procure.config.CommonThreadPoolConfig;
import com.publicisgroupe.procure.mvp.procure.config.constant.BudweiserConst;
import com.publicisgroupe.procure.mvp.procure.domain.bo.sys.SystemInfoBO;
import com.publicisgroupe.procure.mvp.procure.domain.entity.SysOperationLog;
import com.publicisgroupe.procure.mvp.procure.domain.vo.budweiser.TokenVO;
import com.publicisgroupe.procure.mvp.procure.manager.SysOperationLogManager;
import com.publicisgroupe.procure.mvp.procure.utils.BudweiserUtil;
import com.publicisgroupe.procure.mvp.procure.utils.SystemUtils;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.expression.EvaluationContext;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;
import java.text.MessageFormat;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.Stream;

@Aspect
@Slf4j
@Component
@RequiredArgsConstructor
public class OperationLogAspect {

    private final SysOperationLogManager operationLogManager;

    @Pointcut("@annotation(com.publicisgroupe.procure.mvp.common.core.annotation.OperationLog)")
    public void sysOperationLog() {
    }

    @Before("sysOperationLog()")
    public void logOperation(JoinPoint joinPoint) {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        CommonThreadPoolConfig.EXE_LOG_TASK.execute(() -> {
            TokenVO tokenVO = BudweiserUtil.getTokenDefault(request.getHeader(BudweiserConst.TOKEN));
            SystemInfoBO systemInfo = SystemUtils.getSystemInfo(request);
            log.info("aop before systemInfo:{}", systemInfo);
            MethodSignature signature = (MethodSignature) joinPoint.getSignature();
            Method method = signature.getMethod();
            Object[] args = joinPoint.getArgs();
            String[] parameterNames = signature.getParameterNames();
            ExpressionParser parser = new SpelExpressionParser();
            EvaluationContext context = new StandardEvaluationContext();
            if (parameterNames != null) {
                for (int i = 0; i < parameterNames.length; i++) {
                    context.setVariable(parameterNames[i], args[i]);
                }
            }
            SysOperationLog log = new SysOperationLog();
            OperationLog operationLog = method.getAnnotation(OperationLog.class);
            log.setDescription(operationLog.desc());
            String[] params = operationLog.params();
            if (params != null && params.length > 0) {
                String[] strValueArr = new String[params.length];
                for (int i = 0; i < params.length; i++) {
                    Expression exp = parser.parseExpression(params[i]);
                    String value = exp.getValue(context, String.class);
                    strValueArr[i] = value;
                }
                String description = MessageFormat.format(operationLog.desc(), strValueArr);
                log.setDescription(description);
            }
            String idKey = operationLog.idKey();
            if (StrUtil.isNotBlank(idKey)) {
                Expression exp = parser.parseExpression(idKey);
                Object value = exp.getValue(context);
                if (value instanceof List) {
                    log.setDataIds((List<Long>) value);
                } else {
                    String forceValue = String.valueOf(value);
                    log.setDataIds(Stream.of(Long.valueOf(forceValue)).collect(Collectors.toList()));
                }
            }
            log.setOperatorId(tokenVO.getUserDTO().getId().intValue());
            log.setOperator(tokenVO.getUserDTO().getStaffName());
            log.setIpAddress(systemInfo.getIpv4Long());
            log.setModule(operationLog.module());
            log.setType(operationLog.type());
            operationLogManager.save(log);
        });
    }

}
```

