# mob-boot-debug组件-优雅的输出你的调用链


##背景介绍
在微服务架构下，系统多，调用链条长，最上层系统 -> 中台系统 -> 底层系统，系统越复杂，调用链条越深，问题分析定位涉及多系统越多。
无线作为系统公司系统架构的最顶层，依赖非常多的中后台系统，通过rest、tsp、pebble调用，聚合组装处理数据，返回给前端pc，app，m站，
小程序，第三方应用等，然后呈现给用户。依赖的接口多，任何一个接口出问题，都可能影响前端的数据的正确性或者呈现，
分析定位问题比较麻烦，前端涉及的业务多，技术支持多，技术支持处理需要考虑及时响应。于是mob-boot-debug应运而生，
通过mob-boot-debug组件可以优雅的输出依赖接口的调用链及其相关数据。
##设计目标
1、**提高开发／测试质量效率**  
* 能简单清晰地看到接口依赖，依赖接口的性能，请求参数和返回。
* 接口重复调用／不合理调用参数／不合理缓存设置问题直接可见。
* 提高开发效率和质量。
* 方便测试定位分析问题
2、**方便技术支持**
* 节省时间：有工具后，数据及其依赖问题，单系统1分钟内就能定位到问题，一个技术一个系统节省半小时（保守），一个问题涉及三个系统，可节省2个小时左右。
* 不熟悉代码也能处理：只要熟悉业务，数据来源即可，比如产品经理，测试。
* 单方处理：只需要产品、前端、后端一方就能做技术支持了，之前必须要前后端一起才能完成。
* 能够直接复现用户线上的问题和数据
* 工具支持刷新单次接口调用依赖的缓存
3、**工具使用简单方便**
* 使用简单方便，点点鼠标就能解决
* 通过组件方式，方便应用接入
## 具体实现
1、**思路**  

* 通过url中加入*/debug/*启动输出调用链的功能
* 利用Spring的AOP切面功能，通过切面可以记录tsp、rest、pebble的出入参记录到RequestLog对象中
* 继承AbstractHttpMessageConverter,将我们记录在RequestLog对象中的调用链作为出参返回
<img src="http://m.tuniucdn.com/fb2/t1/G5/M00/6C/37/Cii-slxTNU-IXN5dAACjLwa7d2gAATrNgPi-oAAAKNH039.jpg" width="600" hegiht="413" align=center />

2、**代码实现**
* RequestLog,调用日志类，这个类是用来记录我们的调用链的日志的(调用的依赖放，调用的出入参，调用的请求时间)，这个类同时也是一个spring bean，
代码中的@Scope("request")表明这个bean在每次的http请求中都会new一次。这样就让我们对restClient,tspClient,pebbleClient做切面时可以获取
一个全局的对象用来存储每次的调用日志,避免线程不安全，这个对象add日志操作做同步处理
```java
@Component
@Scope("request")
public class RequestLogs implements InitializingBean {

    private String requestId;
    private String requestUri;
    private String parameters;
    private Map<String, TspInfo> dependentTsps;
    private long beginMills;
    private List<String> timeingDiagram;
    private Map<String, RedisInfo> keys;
    private boolean ifDebug;
    private boolean noCache;
    private String clientIp;
    private Map<String, PebbleInfo> dependentPebbles;
    
    // ....
    // 同步添加tsp调用日志
    public synchronized void addTspInfo(TspInfo tspInfo, long startMills, long endMills) {
        dependentTsps.put(dependentTsps.size()
                + "-threadId:" + Thread.currentThread().getId()
                + "-duationMills:" + (endMills - startMills)
                + "-" + tspInfo.getTspName(), tspInfo);
        int part1Num = (int) Math.ceil((startMills - beginMills) / 10d);
        int part2Num = (int) Math.ceil((endMills - startMills) / 10d);
        StringBuilder diagPerTsp = new StringBuilder("[" + (dependentTsps.size() - 1) + "]:");
        for (int i = 0; i < part1Num; i++) {
            diagPerTsp.append("=");

        }
        for (int i = 0; i < part2Num; i++) {
            diagPerTsp.append("+");
        }
        timeingDiagram.add(diagPerTsp.toString());
    }
    //...
}
```

* 这里我们用另外一个LogSoldier的静态方法来获取这个RequestLog对象。
```java
public class LogSoldier {
   
    public static RequestLogs getRequestLogs() {
        RequestLogs result;
        try {
            result = SpringTool.getApplicationContext().getBean("requestLogs", RequestLogs.class); // SpringTool 继承ApplicationContextAware 可以从ApplicationContext中获取bean
        } catch (Exception e) {
            result = null;
        }
        if (result != null) {
            return result;
        }
        return null;
    }

}
```
* 从requestMapping开始,这里通过@RequestMapping("/debug/**")来拦截所有/debug/**的请求，对requestLogs写入一些标记，再通过request.getRequestDispatcher(realPath).forward(request, response)
进行内部请求转发,转发/debug/后面的路径下。
```java

@RestController
public class DebugController {
    @RequestMapping("/debug/**")
    public String debug(HttpServletRequest request, HttpServletResponse response, boolean noCache, @RequestData Object body) throws AbstractApiException {
        RequestLogs requestLogs = LogSoldier.getRequestLogs();
        requestLogs.setBeginMills(System.currentTimeMillis());
        requestLogs.setIfDebug(true);
        requestLogs.setNoCache(parseNoCacheFromBody((Map) body, noCache));
        String realPath = requestLogs.getRequestUri().replaceFirst(".*debug", "");
        try {
            request.getRequestDispatcher(realPath).forward(request, response);
        } catch (Exception e) {
            if (e.getCause() != null && e.getCause() instanceof AbstractApiException) {
                throw (AbstractApiException) e.getCause();
            } else if (e.getCause() != null
                    && e.getCause().getCause() != null
                    && e.getCause().getCause() instanceof AbstractApiException) {
                throw (AbstractApiException) e.getCause().getCause();
            }
            LOGGER.error("debug forward error.", e);
        }
        return "Internal server error.";
    }
}

```
* 对tspClient/restClient进行切面,将出入参写入到requestLogs中，对pebbleClient的出入参的记录是通过继承Pebble组件提供的AbstractFilter来实现的。
```java
@Aspect
@Configuration
public class TspAdvice implements ApplicationContextAware {
    //...

   @Around(value = "(execution(public  * com.tuniu.mob.boot.tsp.TuniuTspSyncExecutor.get(..))) " +
               "|| (execution(public  * com.tuniu.mob.boot.tsp.TuniuTspSyncExecutor.post(..))) " ) //... 多个切点表达式
       public Object syncInvoke(ProceedingJoinPoint point) throws Throwable {
           Object[] args = point.getArgs();
           RequestLogs requestLogs = LogSoldier.getRequestLogs();
           if (requestLogs != null && requestLogs.isIfDebug()) {
               Object returnValue = null;
               Throwable throwable = null;
               long startMills = System.currentTimeMillis();
               try {
                   returnValue = point.proceed(args);//方法出参
               } catch (Throwable t) {
                   throwable = t;
               }
               long endMills = System.currentTimeMillis();
               RequestLogs.TspInfo tspInfo = new RequestLogs.TspInfo();
               //写入接口名
               if (args.length > NUM_2) {
                   tspInfo.setRequest(JSON.toJSON(args[NUM_2]));
                   if (args[0] instanceof String) {
                       tspInfo.setTspName((String) args[0]);
                   }
               }
               //写入出参
               if (throwable == null) {
                   tspInfo.setResponse(JSON.toJSON(returnValue));
               }
               //写入历时
               requestLogs.addTspInfo(tspInfo, startMills, endMills);
               if (throwable != null) {
                   throw throwable;
               }
               return returnValue;
           } else {
               return point.proceed(args);//方法出参
           }
       }
       //...
}

```
* 最后这里通过继承AbstractHttpMessageConverter并且注册到spring容器中,实现把requestLogs对象中的内容作为我们接口的出参返回。
```java
public class DebugHttpMessageConverter extends JsonWrapperHttpMessageConverter {
    private static final String RESPONSE = "response";
    private static final String REQUEST_LOGS = "requestLogs";

    @Override
    protected void writeInternal(Object object, HttpOutputMessage outputMessage) throws IOException {
        Map<String, Object> result = new HashMap<>();
        RequestLogs requestLogs = LogSoldier.getRequestLogs();
        if (requestLogs != null && requestLogs.isIfDebug()) {
            outputMessage.getHeaders().setContentType(MediaType.APPLICATION_JSON);
            byte[] encodedParam;
            if (!(object instanceof ErrorDataResponse)) {
                DefaultResponse response = new DefaultResponse();
                response.setData(object);
                response.setErrorCode(DefaultResponse.OK_CODE);
                response.setMsg("OK");
                response.setSuccess(true);
                result.put(RESPONSE, response);
            } else {
                result.put(RESPONSE, object);
            }
            result.put(REQUEST_LOGS, requestLogs);
            encodedParam = parseToByte(result);
            outputMessage.getBody().write(encodedParam);
        } else {
            super.writeInternal(object, outputMessage);
        }
    }
}

```
## 项目接入

* 项目框架是 spring boot 
* 项目中引入 mob-boot-debug组件 compile("com.tuniu.mob:mob-boot-debug:${mobBootVersion}")


## 使用示例
* 如下图，在请求路径中加入了/debug,可以输出最后的response 和所有的调用链日志  
<img src="http://m.tuniucdn.com/fb2/t1/G4/M00/B0/C0/Cii-VVxTLzGIEsCvAASCiQFbZaEAADqgwNBBwwABIKh290.png" width="600" hegiht="413" align=center />

## 小提示
* 结合前端的toolbar组件结合起来使用会更加方便。  

<img src="http://m.tuniucdn.com/fb2/t1/G5/M00/6D/91/Cii-tFxT-muILDugAA2rZytF24gAATr3AIx7LkADat_111.png" width="600" hegiht="413" align=center />

<img src="http://m.tuniucdn.com/fb2/t1/G4/M00/B1/1F/Cii-VVxT--mIcdPvAAeCQlXf4IIAADrRAHds20AB4Ja892.png" width="600" hegiht="413" align=center />



## 写在最后
* 不足之处: 1.对于自定义的线程异步调用不支持；2.spring mvc的项目无法直接使用；

* 注意事项: 对于幂等的接口，如查询的接口可以通过debug组件去输出调用链的日志，非幂等的接口，如一些写操作的接口谨慎使用