# SpringMVC



1. @RequestMapping

        - value : request url path, eg: "test" or "/test"
        - method: request method, eg: RequestMethod.GET, RequestMethod.POST, RequestMethod.PUT, RequestMethod.DELETE
        - params: {} set request parameters (data sent from client to server), support expression
        - 放在类上，相当于多添加了一层访问路径
        - 里面的参数越多，对request mapping的约束越多

2. @PathVariable 

![pathVariable](imagePool/pathVariable.png)



3. 如果利用表单发送put 或 delete请求，须要做: 
    
        - 将HiddenHttpMethodFilter 添加到 web.xml
![hiddenHttpMethodFilter](imagePool/hiddenHttpMethodFilter.png)

        - form表单中添加隐藏域 name="_method" value="PUT" 或 ”DELETE"
![putDeleteFormRequest](imagePool/putDeleteFormRequest.png)


### Note: 如果使用ajax 发送请求就无须添加 HiddenHttpMethodFilter了，因为ajax自带8种请求方法的方式； 而form表单中只有两种请求方式(get / post), 无法发送put 或 delete 的请求，因此需要HiddenHttpMethodFilter，在servlet之前进行过滤 ~



4. 获取客户端的数据
    
        POST request: 
![formPostRequest](imagePool/formPostRequest.png)

        方式 1: 在请求的方法中，加入响应的形参，保证形参的参数名和传递数据的参数名保持一致就可以自动赋值
![getParameterByParameterName](imagePool/getParameterByParameterName.png)
    
        方式 2: 使用springMVC注解
    
            @RequestParam("xxx")String xxx 来接收参数
                - value: 当形参名和数据参数名不一致时，用value指定数据参数
                - required: 设置参数是否必须赋值，默认为true, 若设置为false， 则无需强制赋值
                - defaultValue: 若形参获得的值为null, 则可设置一个默认值
![getParameterByRequestParam](imagePool/getParameterByRequestParam.png)

            @RequestParam(value="xxx", required=true/false, defaultValue="xxx")String xxx 获取请求头信息
        
            @CookieValue(value="xxx", required=true/false, defaultValue="xxx")String xxx 获取cookie信息
            
        方式 3: 使用一个 pojo 作为形参获取客户端数据，要求实体类中的属性名一定要和表单元素的数据属性名一致，且支持级联赋值 
![getParameterByPojo](imagePool/getParameterByPojo.png)
        
        方式 4: 使用 javax.servlet原生 HttpServletRequest作为形参
![getParameterByHttpServletRequest](imagePool/getParameterByHttpServletRequest.png)




5. 作用域放值 + 页面跳转

        方式 1: ModelAndView
            实现原理 (source code): 给request域存值 （request.setAttribute(key, value) + 页面跳转 (request.getRequestDispacher().forward(request, response)
![modelAndView](imagePool/modelAndView.png)


        方式 2: Map<String Object> as parameter
![mapView](imagePool/mapView.png)
        
        
        方式 3: Model model as parameter
![modelView](imagePool/modelView.png)

    
        方式 4: 使用 javax.servlet原生 HttpServletRequest作为形参
![httpServletRequestSetAttributeView](imagePool/httpServletRequestSetAttributeView.png)




6. 处理静态资源请求/访问

        - 在springMVC的配置文件中添加: (Tomcat中的default servlet) 
![staticResourceDefaultServlet](imagePool/staticResourceDefaultServlet.png)

        - 当Tomcat default servlet 所设置的<url-pattern>的值与开发人员所配置的servlet的<url-pattern>的值
        相同时 (<url-pattern>/</url-pattern>)，以开发人员所配置的优先进行匹配; 因此先通过DispatcherServlet
        进行处理，有则处理，无则交给Tomcat DefaultServlet处理。



7. mvc驱动标签
    
![mvcDriven](imagePool/mvcDriven.png)

        - 启动Tomcat Default Handler 处理静态资源请求
        - 启动 jackson 将java 对象转化成 json
        
        


8. JSON  操作

        - types: 
            - json object: {key1: val1, key2: val2}
            - json array: [{key1: val1, key2: val2}, {key3: val3, key4: val4}]
            
        - SpringMVC 对JSON的支持: 直接返回Java object
            - step 1: 开启 MVC 驱动
![mvcDriven](imagePool/mvcDriven.png)

           - step2: 添加 Jackson jars
![jacksonJars](imagePool/jacksonJars.png)

           - step3: 添加 @ResponseBody 在@controller方法上
![responseBodyAnnotation](imagePool/responseBodyAnnotation.png)

           - step4: 把要返回给客户端的数据直接作为返回值返回 (obj, map, list)



9. Filter vs Interceptor

        1. 实现原理不同:
            - Filter 是基于函数回调
            - Interceptor 是基于Java的反射机制 (动态代理)
            
        2. 使用范围不同
            - Filter 实现的是 javax.servlet.Filter 接口，而这个接口是在 Servlet
            规范中定义的，因此 Filter 的使用要依赖于 Tomcat 等容器，导致他只能在 web
            程序中使用
            
            - Interceptor 是 Spring的一个组件，由 Spring 容器管理，并不依赖Tomcat等
            web 容器，是可以单独使用的。不仅能应用在 web 程序中，也可以应用在普通的
            Application 程序中
            
        3. 触发机制不同 （执行顺序)
        
![filterInterceptorExecutionOrder](imagePool/filterInterceptorExecutionOrder.png)

            - Filter 是在请求进入容器后，但在进入 Servlet 之前进行预处理，请求结束是在 servlet 处理完之后
            - Interceptor 是在请求进入 Servelt 后，在进入 Controller 之前进行预处理的，Controller中渲染了对应的视图之后结束


10. Interceptor 拦截器

        0. 实现方式： 
            - implement HanderInterceptor 接口

        1. 各个方法在控制器中的处理器中的作用点 (AOP 思想)
[interceptor.png](imagePool/interceptor.png)

      2. 规则
          - return true -- 放行； return false -- 拦截
          - preHandle() 和 postHandle()/afterCompletion() 对拦截器的遍历顺序正好相反: preHnadle()是从小到大, postHandle() 是从大到小, afterCompletion() 是从大到小
          - 一旦被某个拦截器拦截成功后:
                - 后面所有拦截器的preHandle()方法都不会执行
                - 所有拦截器postHanle()方法都不会执行
                - afterCompletion()方法会被触发，执行顺序为: 当前拦截器的上一个拦截器.afterCompletion() 到第0个拦截器.afterCompletion()

        
