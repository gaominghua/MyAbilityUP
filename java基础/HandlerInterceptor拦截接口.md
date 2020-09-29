### HandlerInterceptor拦截接口
```java
public interface HandlerInterceptor {

/**预处理回调方法，实现处理器的预处理（如检查登陆），第三个参数为响应的处理器，自定义Controller
返回值：
true表示继续流程（如调用下一个拦截器或处理器）；
false表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器，此时我们需要通过response来产生响应；
**/
boolean preHandle(HttpServletRequest request, HttpServletResponse response,
Object handler)

throws Exception;

/**后处理回调方法，实现处理器的后处理（但在渲染视图之前），此时我们可以通过modelAndView（模型和视图对象）对模型数据进行处理或对视图进行处理，modelAndView也可能为null**/


void postHandle(

HttpServletRequest request, HttpServletResponse response, Object handler,
ModelAndView modelAndView)

throws Exception;

/\*\*

\*
整个请求处理完毕回调方法，即在视图渲染完毕时回调，如性能监控中我们可以在此记录结束时间并输出消耗时间，还可以进行一些资源清理，类似于try-catch-finally中的finally，但仅调用处理器执行链中

\*/

void afterCompletion(

HttpServletRequest request, HttpServletResponse response, Object handler,
Exception ex)

throws Exception;

}

preHandle

调用时间：Controller方法处理之前

执行顺序：链式Intercepter情况下，Intercepter按照声明的顺序一个接一个执行

若返回false，则中断执行，**注意：不会进入afterCompletion**

postHandle

调用前提：preHandle返回true

调用时间：Controller方法处理完之后，DispatcherServlet进行视图的渲染之前，也就是说在这个方法中你可以对ModelAndView进行操作

执行顺序：链式Intercepter情况下，Intercepter**按照声明的顺序倒着执行**。

备注：postHandle虽然post打头，但post、get方法都能处理

afterCompletion

调用前提：preHandle返回true

调用时间：DispatcherServlet进行视图的渲染之后

多用于清理资源

**拦截器适配器HandlerInterceptorAdapter**  
有时候我们可能只需要实现三个回调方法中的某一个，如果实现HandlerInterceptor接口的话，三个方法必须实现，不管你需不需要，此时spring提供了一个HandlerInterceptorAdapter适配器（种适配器设计模式的实现），允许我们只实现需要的回调方法。

public abstract class HandlerInterceptorAdapter implements
AsyncHandlerInterceptor {

/\*\*

\* 默认是true

\*/

\@Override

public boolean preHandle(HttpServletRequest request, HttpServletResponse
response, Object handler)

throws Exception {

return true;

}

/\*\*

\* This implementation is empty.

\*/

\@Override

public void postHandle(

HttpServletRequest request, HttpServletResponse response, Object handler,
ModelAndView modelAndView)

throws Exception {

}

/\*\*

\* This implementation is empty.

\*/

\@Override

public void afterCompletion(

HttpServletRequest request, HttpServletResponse response, Object handler,
Exception ex)

throws Exception {

}

/\*\*

\*
不是HandlerInterceptor的接口实现，是AsyncHandlerInterceptor的，AsyncHandlerInterceptor实现了HandlerInterceptor

\*/

\@Override

public void afterConcurrentHandlingStarted(

HttpServletRequest request, HttpServletResponse response, Object handler)

throws Exception {

}

}

这样在我们业务中比如要记录系统日志，日志肯定是在afterCompletion之后记录的，否则中途失败了，也记录了，那就扯淡了。一定是程序正常跑完后，我们记录下那些对数据库做个增删改的操作日志进数据库。所以我们只需要继承HandlerInterceptorAdapter，并重写afterCompletion一个方法即可，因为preHandle默认是true。
```
![测11111试](https://s0.2mdn.net/10007680/PROGBANNER_SKY-SCRAP_SG1.jpg)