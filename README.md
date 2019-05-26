# springboot笔记 #

## 搭建环境工具
sts，右击，选中Web，DevTools

## 配置文件
	
<span class="stress">springboot中，哪些参数可以进行自定义配置，配置参数是什么，在application.properties配置文件中，自定义参数的格式什么样的，我们可以参考spring-boot-autoconfigure-2.0.3.RELEASE.jar。以配置数据源为例子：<br>

		@ConfigurationProperties(prefix = "spring.datasource")
		public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean {
			private String driverClassName;

			/**
			 * JDBC URL of the database.
			 */
			private String url;
		
			/**
			 * Login username of the database.
			 */
			private String username;
		
			/**
			 * Login password of the database.
			 */
			private String password;
		}
<span class="stress">我们在application.properties配置文件中，以spring.datasource作为key的前缀，我们配置参数时，key为	spring.datasource.url=
</span>


- **置文件的生效顺序，会对值进行覆盖：**
	1. @TestPropertySource 注解
	2. 命令行参数
	3. Java系统属性（System.getProperties()）
	4. 操作系统环境变量
	5. 只有在random.*里包含的属性会产生一个RandomValuePropertySource
	6. 在打包的jar外的应用程序配置文件（application.properties，包含YAML和profile变量）
	7. 在打包的jar内的应用程序配置文件（application.properties，包含YAML和profile变量）
	8. 在@Configuration类上的@PropertySource注解
	9. 默认属性（使用SpringApplication.setDefaultProperties指定 application.properties）

- **配置随机值**

	1. roncoo.secret=${random.value}
	2. roncoo.number=${random.int}
	3. roncoo.bignumber=${random.long}
	4. roncoo.number.less.than.ten=${random.int(10)}
	5. roncoo.number.in.range=${random.int[1024,65536]}
	
	读取使用注解：@Value(value = "${roncoo.secret}")  注：出现黄点提示，是要提示配置元数据，可以不配置

- **属性占位符**
	
	当application.properties里的值被使用时，它们会被存在的Environment过滤，所以你能够引用先前定义的值（比如，系统属性）。
	roncoo.name=www.roncoo.com  </br>
	roncoo.desc=${roncoo.name} is a domain name

- **Application属性文件，按优先级排序，位置高的将覆盖位置低的**
	1. 当前目录下的一个/config子目录
	2. 当前目录
	3. 一个classpath下的/config包
	4. classpath根路径（root）<br>
	这个列表是按优先级排序的（列表中位置高的将覆盖位置低的）<br>

- **配置应用端口和其他配置的介绍**

	端口配置：server.port=8090 <br>
	时间格式化：spring.jackson.date-format=yyyy-MM-dd HH:mm:ss <br>
	时区设置：spring.jackson.time-zone=Asia/Chongqing

- **使用YAML代替Properties**
	注意写法：冒号后要加个空格

## 日志文件配置
- spring boot默认会加载**classpath:logback-spring.xml**或者**classpath:logback-spring.groovy**

- 使用自定义配置文件，在application.properties中配置方式为：
	logging.config=classpath:logback-roncoo.xml
	注意：不要使用logback这个来命名，否则spring boot将不能完全实例化

- 日志文件配置

	```


		<?xml version="1.0" encoding="UTF-8"?>
		<configuration>
	
			<!-- 文件输出格式 -->
			<property name="PATTERN" value="%-12(%d{yyyy-MM-dd HH:mm:ss.SSS}) |-%-5level [%thread] %c [%L] -| %msg%n" />
			<!-- test文件路径 -->
			<property name="TEST_FILE_PATH" value="c:/opt/roncoo/logs" />
			<!-- pro文件路径 -->
			<property name="PRO_FILE_PATH" value="/opt/roncoo/logs" />

			<!-- 开发环境 -->
			<springProfile name="dev">
				<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
					<encoder>
						<pattern>${PATTERN}</pattern>
					</encoder>
				</appender>
				
				<logger name="com.roncoo.education" level="debug"/>
		
				<root level="info">
					<appender-ref ref="CONSOLE" />
				</root>
			</springProfile>

			<!-- 测试环境 -->
			<springProfile name="test">
				<!-- 每天产生一个文件 -->
				<appender name="TEST-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
					<!-- 文件路径 -->
					<file>${TEST_FILE_PATH}</file>
					<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
						<!-- 文件名称 -->
						<fileNamePattern>${TEST_FILE_PATH}/info.%d{yyyy-MM-dd}.log</fileNamePattern>
						<!-- 文件最大保存历史数量 -->
						<MaxHistory>100</MaxHistory>
					</rollingPolicy>
					
					<layout class="ch.qos.logback.classic.PatternLayout">
						<pattern>${PATTERN}</pattern>
					</layout>
				</appender>
				
				<root level="info">
					<appender-ref ref="TEST-FILE" />
				</root>
			</springProfile>

			<!-- 生产环境 -->
			<springProfile name="prod">
				<appender name="PROD_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
					<file>${PRO_FILE_PATH}</file>
					<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
						<fileNamePattern>${PRO_FILE_PATH}/warn.%d{yyyy-MM-dd}.log</fileNamePattern>
						<MaxHistory>100</MaxHistory>
					</rollingPolicy>
					<layout class="ch.qos.logback.classic.PatternLayout">
						<pattern>${PATTERN}</pattern>
					</layout>
				</appender>
				
				<root level="warn">
					<appender-ref ref="PROD_FILE" />
				</root>
			</springProfile>

		</configuration>
	```

## 模板引擎 ##
一．	**spring boot的web应用开发，是基于spring mvc**

二．	**Spring boot 在spring默认基础上，自动配置添加了以下特性：**

1. 包含了ContentNegotiatingViewResolver和BeanNameViewResolver beans。
2.	对静态资源的支持，包括对WebJars的支持。
3.	自动注册Converter，GenericConverter，Formatter beans。
4.	对HttpMessageConverters的支持。
5.	自动注册MessageCodeResolver。
6.	对静态index.html的支持。
7.	对自定义Favicon的支持。
8.	主动使用ConfigurableWebBindingInitializer bean

三．**模板引擎的选择**

	FreeMarker<br>
	Thymeleaf<br>
	Velocity (1.4版本之后弃用，Spring Framework 4.3版本之后弃用)<br>
	Groovy<br>
	Mustache<br>
注：jsp应该尽量避免使用，原因如下：<br>

1.	jsp只能打包为：war格式，不支持jar格式，只能在标准的容器里面跑（tomcat，jetty都可以） 
2.	内嵌的Jetty目前不支持JSPs
3.	Undertow不支持jsps
4.	jsp自定义错误页面不能覆盖spring boot 默认的错误页面

### FreeMarker模板引擎 ###
- **jar包依赖**
 
		<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-freemarker</artifactId>
			</dependency>
			
			<!-- 对jquery的依赖-->
			<dependency>
				<groupId>org.webjars</groupId>
				<artifactId>jquery</artifactId>
				<version>2.1.4</version>
		</dependency>
- **静态文件依赖**
	
	springboot自动加载static目录下的静态文件

- **静态页面文件格式**

	index.ftl
	
	
			<!DOCTYPE html>
			<html>
			<head>
			<meta charset="UTF-8">
			<title>Insert title here</title>
			<link rel="stylesheet" type="text/css" href="/css/index/index.css" />
			<link rel="shortcut icon" href="/images/favicon.ico">
			<script type="text/javascript" src="/webjars/jquery/2.1.4/jquery.js"></script>
			<script type="text/javascript">
				$(function(){
					$("p").click(function(){
						alert("点击了");
					})
				})
				
			</script>
			</head>
			<body>
				<img src="/images/photo111.png">
				<p id="name">${name}</p>
			</body>
			</html>

### Thymeleaf模板引擎 ###

- jar包依赖

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>

- 文件格式
	
	index.html
	
		<!DOCTYPE html>
		<html xmlns="http://www.w3.org/1999/xhtml"
		      xmlns:th="http://www.thymeleaf.org">
			<head>
				<meta charset="UTF-8">
				<title>Insert title here</title>
				<link rel="stylesheet" type="text/css" href="/css/index/index.css" />
				<link rel="shortcut icon" href="/images/favicon.ico">
				<script type="text/javascript" src="/webjars/jquery/2.1.4/jquery.js"></script>
				<script type="text/javascript">
					$(function(){
						$("p").click(function(){
							alert("点击了");
						})
					})
					
				</script>
			</head>
			<body>
				<img src="/images/photo111.png">
				<p id="name" th:text="${name}"></p>
			</body>
		</html>

	**在html标签里加上thymeleaf的约束: <br>**
	xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"

### jsp模板引擎 ###

- jar包依赖


		<dependency>
			<groupId>org.apache.tomcat.embed</groupId>
			<artifactId>tomcat-embed-jasper</artifactId>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
		</dependency>

- 打包格式必须为war包

- 工程结构
	
	

	![](https://i.imgur.com/x2XFXQX.png)

- 配置文件添加参数
	- spring.mvc.view.prefix=/WEB-INF/index/
	- spring.mvc.view.suffix=.jsp


## 异常处理

springboot项目运行过程中出现异常，springboot默认将异常信息处理交由BasicErrorController处理，映射路径为error

- **Spring Boot 将所有的错误默认映射到/error， 实现ErrorController**

	自定义一个异常控制类，覆盖原来的，可以根据请求类型将异常处理分发到不同的处理方法中进行处理。可以同时实现返回错误页面，也可以返回json数据


		/**
		 * 实现ErrorController接口，覆盖掉springboot默认的/error请求路径
		 * @author sxq
		 * @time 2018年6月6日 上午10:58:30
		 *
		 */
		@Controller
		@RequestMapping("error")
		public class MyErrorController implements ErrorController {
		
			
			@Autowired
		    private ErrorAttributes errorAttributes;
			
			@Override
			public String getErrorPath() {
				return "/error";
			}
			
			
			/**
			 * 	如果是页面请求，则进入该方法中，返回页面
			 *	@param request
			 *	@return
			 *	Map<String,Object>
			 */
			@RequestMapping(produces=MediaType.TEXT_HTML_VALUE)
			public String error(HttpServletRequest request){
				Map<String, Object> map = getErrorAttributes(request, false);
				Integer status = (Integer) map.get("status");
				if(404==status){
					return "error/404";
				}else if(500==status){
					return "error/500";
				}
				return "error/other";
				
		//		
			}
			
			/**
			 * 	如果json请求，进入该方法中，返回json数据
			 *	@param request
			 *	@return
			 *	String
			 */
			@RequestMapping(produces=MediaType.APPLICATION_JSON_VALUE)
			@ResponseBody
			public Map<?, ?> error1(HttpServletRequest request){
				Map<String, Object> map = new HashMap<String, Object>();
				//自定义异常类，获取异常信息
				Throwable throwable= getThrowable(request);
				if(throwable instanceof MyException){
					MyException exception = (MyException) throwable;
					map.put("code", exception.getCode());
					map.put("message", exception.getMessage());
				}else{
					map.put("code", "999");
					map.put("message", "未知错误");
				}
				return map;
			}
		
			/**
			 * 	获取异常信息集合
			 *	@param request
			 *	@param includeStackTrace 是否获取栈信息，即报错的位置
			 *	@return
			 *	Map<String,Object>
			 */
		   private Map<String, Object> getErrorAttributes(HttpServletRequest request, boolean includeStackTrace) {
			   WebRequest webRequest = new ServletWebRequest(request);
		        return errorAttributes.getErrorAttributes(webRequest, includeStackTrace);
		    }
		   
		   
		   /**
		    * 	获取异常类
		    *	@param request
		    *	@return
		    *	Throwable
		    */
		   private Throwable getThrowable(HttpServletRequest request){
			   WebRequest webRequest = new ServletWebRequest(request);
			   return errorAttributes.getError(webRequest);
		   }
		
		
		}
	`
- **添加自定义错误页面**
	1. html静态页面
	
		可以在resources/templates/error目录下定义，也可以在/resources/public/error定义，前者的优先级高于后者

	![](https://i.imgur.com/letXaBp.png)
	2. 模板引擎下配置
	
		可以在resources/templates/error目录下定义，也可以在/resources/public/error定义，前者的优先级高于后者
		.html结尾的文件可以放在/resources/public/error目录下，但.ftl结尾的文件只能放在resources/templates/error目录下
	
	![](https://i.imgur.com/BncWLxW.png)


- **使用@ControllerAdvice注解处理异常**

		@ControllerAdvice
		public class MyExceptionHandler {
			
			/**
			 * 运行时异常
			 *	@param exception
			 *	@return
			 *	ModelAndView
			 */
			@ExceptionHandler({ RuntimeException.class })
			@ResponseStatus(HttpStatus.OK)
			public ModelAndView processException(RuntimeException exception) {
				System.out.println("进来了");
				ModelAndView m = new ModelAndView();
				m.addObject("roncooException", exception.getMessage());
				m.setViewName("error/500");
				return m;
			}
			
			/**
			 * 定义全局异常
			 *	@param exception
			 *	@return
			 *	ModelAndView
			 */
			@ExceptionHandler({ Exception.class })
			@ResponseStatus(HttpStatus.OK)
			public ModelAndView processException(Exception exception) {
				ModelAndView m = new ModelAndView();
				m.addObject("roncooException", exception.getMessage());
				m.setViewName("error/500");
				return m;
			}
		
		}

	找/thymeleaf/src/main/resources/templates/error目录下的错误页面


## springboot 自定义过滤器，监听器，servlet ##

### 第一种方式：使用@ServletComponentScan注解 ###

	@SpringBootApplication(scanBasePackages={"com.example.controller","com.example.exception","com.example.config"})
	@ServletComponentScan(basePackages={"com.example.filter","com.example.listener"})
	public class ThymeleafApplication {
	
		public static void main(String[] args) {
			SpringApplication.run(ThymeleafApplication.class, args);
		}
	}

在ThymeleafApplication上加上**ServletComponentScan注解**。

- 自定义过滤器
	使用注解**@WebFilter**(urlPatterns="/*")，urlPatterns="/*"拦截所以请求
	
		@WebFilter(urlPatterns="/*")
		public class MyFilter implements Filter {
		
			private static final LoggerUtil<MyFilter> logger = LoggerUtil.getInstance(MyFilter.class);
			
			@Override
			public void init(FilterConfig filterConfig) throws ServletException {
				logger.info("【MyFilter初始化了】");
			}
		
			@Override
			public void doFilter(ServletRequest request, ServletResponse response,
					FilterChain chain) throws IOException, ServletException {
				HttpServletRequest req = (HttpServletRequest) request;
				String url = req.getRequestURL().toString();
				String uri = req.getRequestURI();
				logger.info("【url】----{0}",url);
				logger.info("【uri】----{0}",uri);
				chain.doFilter(request, response);
			}
		
			@Override
			public void destroy() {
				logger.info("【MyFilter销毁了】");
			}
	
		}

- 自定义监听器
	
	使用注解**@WebListener**
	
		/**
		 * 项目启动的时候，先启动ServletContextListener
		 * @author sxq
		 * @time 2018年6月7日 下午5:54:55
		 *
		 */
		@WebListener
		public class MyListener implements ServletContextListener{
		
			private static final LoggerUtil<MyListener> logger = LoggerUtil.getInstance(MyListener.class);
			@Override
			public void contextInitialized(ServletContextEvent sce) {
				
				logger.info("初始化servlet上下文");
			}
		
			@Override
			public void contextDestroyed(ServletContextEvent sce) {
				
			}
		
		}


- 自定义servlet

	使用注解**@WebServlet**(urlPatterns="/print.do")，url路径前必须加上"/"

		@WebServlet(urlPatterns="/print.do")
		public class MyServlet extends HttpServlet {
		
			/**
			 * serialVersionUID
			 */
			private static final long serialVersionUID = 1L;
			
			private static final LoggerUtil<MyServlet> logger = LoggerUtil.getInstance(MyServlet.class);
		
			@Override
			protected void doGet(HttpServletRequest req, HttpServletResponse resp)
					throws ServletException, IOException {
				Map<String, Object> map = new HashMap<String, Object>();
				map.put("timestamp", System.currentTimeMillis());
				map.put("status", "OK");
				resp.getWriter().write(JSONObject.toJSONString(map, true));
			}
		
			@Override
			protected void doPost(HttpServletRequest req, HttpServletResponse resp)
					throws ServletException, IOException {
				super.doPost(req, resp);
			}
		
			@Override
			public void destroy() {
				logger.info("MyServlet销毁了。。。。。。。。");
				super.destroy();
			}
		
			@Override
			public void init(ServletConfig config) throws ServletException {
				logger.info("MyServlet初始化了。。。。。。。。");
				super.init(config);
			}
		}

### 第二种方式：通过注册 ServletRegistrationBean、 FilterRegistrationBean 和 ServletListenerRegistrationBean 获得控制  ###

- 创建MyServlet extends HttpServlet、MyFilter implements Filter、MyListener implements ServletContextListener
- 在启动类中，使用@Bean注解，注册 ServletRegistrationBean、 FilterRegistrationBean 和 ServletListenerRegistrationBean,从而实现自定义Filter、servlet、listener <br>

		@SpringBootApplication(scanBasePackages={"com.example.controller","com.example.exception","com.example.config"})
		@ServletComponentScan(basePackages={"com.example.filter","com.example.listener","com.example.servlet"})
		public class ThymeleafApplication {
		
			public static void main(String[] args) {
				SpringApplication.run(ThymeleafApplication.class, args);
			}
			
			/**
			 * 注册ServletRegistrationBean
			 *	@return
			 *	ServletRegistrationBean<MyServlet>
			 */
			 @Bean 
			 public ServletRegistrationBean<MyServlet> servletRegistrationBean() { 
			  return new ServletRegistrationBean<MyServlet>(new MyServlet(), "/print.do"); 
			 } 
			
			 /**
			  * 注册FilterRegistrationBean
			  *	@return
			  *	FilterRegistrationBean<MyFilter>
			  */
			 @Bean 
			 public FilterRegistrationBean<MyFilter> filterRegistrationBean() { 
			  return new FilterRegistrationBean<MyFilter>(new MyFilter(), servletRegistrationBean()); 
			 } 
			 
			 /**
			  * 注册ServletListenerRegistrationBean
			  *	@return
			  *	ServletListenerRegistrationBean<MyListener>
			  */
			 @Bean
			 public ServletListenerRegistrationBean<MyListener> servletListenerRegistrationBean(){
				 
				 return new ServletListenerRegistrationBean<MyListener>(new MyListener());
			 }
			 
		}


### 启动类实现ServletContextInitializer接口，直接完成注册。


	@SpringBootApplication(scanBasePackages={"com.example.controller","com.example.exception","com.example.config"})
	@ServletComponentScan(basePackages={"com.example.filter","com.example.listener","com.example.servlet"})
	public class ThymeleafApplication implements ServletContextInitializer{
	
		public static void main(String[] args) {
			SpringApplication.run(ThymeleafApplication.class, args);
		}
	
		@Override
		public void onStartup(ServletContext servletContext)
				throws ServletException {
			servletContext.addFilter("myFilter", new MyFilter()).addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST), true, "/*");
			servletContext.addServlet("myServlet", new MyServlet()).addMapping("/print.do");
			servletContext.addListener(new MyListener());
		}
	}

## 添加拦截器

- 自定义拦截器，继承HandlerInterceptorAdapter


		public class Intercepter extends HandlerInterceptorAdapter {
		
			private static final LoggerUtil<Intercepter> logger = LoggerUtil.getInstance(Intercepter.class);
			@Override
			public boolean preHandle(HttpServletRequest request,
					HttpServletResponse response, Object handler) throws Exception {
				logger.info("进入前置处理方法！");
				return super.preHandle(request, response, handler);
			}
		
			@Override
			public void postHandle(HttpServletRequest request,
					HttpServletResponse response, Object handler,
					ModelAndView modelAndView) throws Exception {
				logger.info("进入后置处理方法");
				super.postHandle(request, response, handler, modelAndView);
			}
		
			@Override
			public void afterCompletion(HttpServletRequest request,
					HttpServletResponse response, Object handler, Exception ex)
					throws Exception {
				// TODO Auto-generated method stub
				super.afterCompletion(request, response, handler, ex);
			}
		
			@Override
			public void afterConcurrentHandlingStarted(HttpServletRequest request,
					HttpServletResponse response, Object handler) throws Exception {
				super.afterConcurrentHandlingStarted(request, response, handler);
			}
		
		}


- 自定义mvc配置类，继承DelegatingWebMvcConfiguration，重写addInterceptors方法

		/**
		 * WebMvcConfigurer配置类
		 * @author sxq
		 * @time 2018年6月7日 下午6:15:32
		 *
		 */
		@Configuration
		public class MvcConfig extends DelegatingWebMvcConfiguration{
			
			private static final LoggerUtil<MvcConfig> logger = LoggerUtil.getInstance(MvcConfig.class);
		
			/**
			 * addResourceHandler----资源处理器需要拦截哪些静态资源请求。
			 * addResourceLocations-----静态资源的位置
			 */
			@Override
			protected void addResourceHandlers(ResourceHandlerRegistry registry) {
				registry.addResourceHandler(new String[]{"/images/**/*","/static/**/*","/templates/**/*"})
					.addResourceLocations("classpath:/images/","classpath:/static/","/templates/");
				super.addResourceHandlers(registry);
			}
			
			/**
			 * addViewController---设置请求url
			 * setViewName----设置返回视图名称
			 */
			@Override
			protected void addViewControllers(ViewControllerRegistry registry) {
				registry.addViewController("/toLogin").setViewName("index/index");
				super.addViewControllers(registry);
			}
			
			/**
			 * addInterceptor---设置拦截器的实例
			 * addPathPatterns---设置拦截器拦截的url
			 */
			@Override
			protected void addInterceptors(InterceptorRegistry registry) {
				logger.info("进入添加拦截器方法上了！");
				registry.addInterceptor(new Intercepter()).addPathPatterns("/**/*.do");
				super.addInterceptors(registry);
			}
			
		}

**注意，在启动类中，扫描该配置类所在的包，使得@configuration注解生效**


## springboot解决CORS（Cross-Origin Resource Sharing, 跨源资源共享）

1. 使用场景

	浏览器默认不允许跨域访问，包括我们平时ajax也是限制跨域访问<br>
	产生跨域访问的情况主要是请求的发起者与请求的接受者的**域名不同，端口不同**

2. 解决方案

	通过设置Access-Control-Allow-Origin来实现跨域访问

3. CORS与jsonp相比的优点
	
	1）JSONP 只能实现 GET 请求，而 CORS 支持所有类型的 HTTP 请求。 <br>
	2）使用 CORS，开发者可以使用普通的 XMLHttpRequest 发起请求和获得数据，比起 JSONP 有更好的
	错误处理。 <br>
	3）JSONP 主要被老的浏览器支持，它们往往不支持 CORS，而绝大多数现代浏览器都已经支持了 CORS <br>

	浏览器支持情况 <br>

			Chrome 3+ 
			Firefox 3.5+ 
			Opera 12+
			Safari 4+
			Internet Explorer 8+

4. springboot 基于jsonp解决跨域问题
	
	服务端实现：
		
		@Controller
		public class JsonpController {
		
			@RequestMapping("/jsonp.do")
			@ResponseBody
			public void jsonp(HttpServletResponse response,@RequestParam String callback){
		//		response.setContentType("text/html");
				response.setCharacterEncoding("utf-8");
				Map<String, Object> map = new HashMap<String, Object>();
				map.put("name", "张三");
				map.put("type", "jsonp");
				try {
					response.getWriter().write(callback+"("+JSONObject.toJSONString(map)+")");
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	
	前端js实现：

		$("#jsonp").click(function(){
			var data = {};
			$.ajax({
				type:"get",
				url:"http://localhost:9080/jsonp.do",
				data:data,
				dataType: "jsonp",
				jsonp:"callback",
				//jsonpCallback:callback,
				success:function(data){
					$("#div").html(JSON.stringify(data));
				}
			})

		});

	<span class="stress">jsonp:</span>在一个jsonp请求中重写回调函数的名字。这个值用来替代在"callback=?"这种GET或POST请求中URL参数里的"callback"部分，比如{jsonp:'onJsonPLoad'}会导致将"onJsonPLoad=?"传给服务器。<br>
	<span class="stress">jsonpCallback:</span>为jsonp请求指定一个回调函数名。这个值将用来取代jQuery自动生成的随机函数名。这主要用来让jQuery生成度独特的函数名，这样管理请求更容易，也能方便地提供回调函数和错误处理。你也可以在想让浏览器缓存GET请求的时候，指定这个回调函数名。



4. 具体实现
	
	
	- 全局配置

		【1】继承**DelegatingWebMvcConfiguration**，重写addCorsMappings(CorsRegistry registry)方法<br>
			<span class="stress">设置请求url映射、设置请求允许来源，允许访问的客户端地址、允许访问请求方法，如GET、POST</span>

	
			@Configuration
			public class MvcConfig extends DelegatingWebMvcConfiguration {
			
				@Override
				protected void addCorsMappings(CorsRegistry registry) {
					/**
					 * addMapping----设置请求url映射
					 * allowedOrigins ----设置请求允许来源，设置客户端地址
					 * allowedMethods -----允许访问请求方法，如GET、POST
					 */
					registry.addMapping("/cors/**").allowedOrigins("http://localhost:8090")
					.allowedMethods("GET");
					super.addCorsMappings(registry);
				}
			
			}

		【2】定义方法
			
			@RestController
			@RequestMapping("/cors")
			public class CorsController extends BaseController{
			
				
				@RequestMapping("/get.do")
				public Map<String, Object> get(){
					Map<String, Object> map = new HashMap<String, Object>();
					map.put("name", "张三");
					map.put("sex", "男");
					logger.debug("跨域返回值{0}", map);
					return map;
				}
			}

		【3】测试js代码示例如下

			$("p").click(function(){
				$.ajax({
					type:"GET",
					url:"http://localhost:9080/cors/get.do",
					success:function(data){
						$("#div").html(JSON.stringify(data));
					},
					
					error:function(data){
						$("#div").html(JSON.stringify(data.responseJSON));
					}
				});
			})


	- 细粒度设置 在方法上使用<span class="stress">@CrossOrigin</span>注解
		
		【1】定义方法
			
			@RestController
			@RequestMapping("/annoCors")
			public class AnnoCorsController extends BaseController{
			
				@RequestMapping("test")
				@CrossOrigin(origins="http://localhost:8090")
				public Map<String, Object> cors(){
					Map<String, Object> map = new HashMap<String, Object>();
					map.put("name", "张三");
					map.put("type", "注解");
					logger.debug("基于注解的跨域返回值{0}", map);
					return map;
				}
			}
		
		
		
		
		
## springboot 文件上传 ##

	springboot文件与springmvc文件上传差不多
- 单个文件上传
	- 前端实现

			<form action="file/upload" enctype="multipart/form-data" method="post">
				<input type="file" name="file" />
				<input type="submit" value="上传单个文件"/>
			</form>

	- 服务端实现

			@RequestMapping("file/upload")
			public String upload(HttpServletRequest request, @RequestParam MultipartFile file){
				File parent = new File("C:\\Users\\admin\\Desktop");
				if(!parent.exists()){
					parent.mkdirs();
				}
				File dir = new File(parent,"b.png");
				try {
					file.transferTo(dir);
				} catch (IllegalStateException e) {
					request.setAttribute("MSG", "文件上传失败");
					e.printStackTrace();
				} catch (IOException e) {
					request.setAttribute("MSG", "文件上传失败");
					e.printStackTrace();
				}
				
				request.setAttribute("MSG", "文件上传成功");
				return "success";
			}

- 批量文件上传

	- 前端实现
		
			<div class="div">
				<form action="uploads" enctype="multipart/form-data" method="post">
					<input type="file" name="file" /><br>
					<input type="file" name="file" /><br>
					<input type="file" name="file" /><br>
					<input type="submit" value="上传多个文件"/>
				</form>
			</div>
	
	- 服务端实现

			@RequestMapping("uploads")
			public @ResponseBody String uploads(MultipartHttpServletRequest request){
				List<MultipartFile> files = request.getFiles("file");
				if(files==null||files.size()==0){
					return "上传文件不能为空！";
				}
				String filePath = request.getSession().getServletContext().getRealPath("/upload");
				File file = new File(filePath);
				if(!file.exists()){
					file.mkdirs();
				}
				boolean flag = true;
				for(MultipartFile multipartFile:files){
					String fileName = multipartFile.getOriginalFilename();
					long size = multipartFile.getSize();
					if(size==0){
						flag = false;
						continue;
					}
					flag = true;
					try {
						multipartFile.transferTo(new File(file, fileName));
					} catch (IllegalStateException | IOException e) {
						e.printStackTrace();
						return "文件上传失败|"+e.getMessage();
					}
				}
				
				if(!flag){
					return "上传文件不能为空！";
				}
				
				return "上传成功";
			}


- 配置文件配置参数
	
	1、spring.http.multipart.maxFileSize=1Mb  上传文件大小<br>
	2、spring.http.multipart.maxRequestSize=100Mb<br>
	3、spring.http.multipart.enabled=true #默认支持文件上传. <br>
	4、spring.http.multipart.file-size-threshold=0 #支持文件写入磁盘. <br>
	5、spring.http.multipart.location= # 上传文件的临时目录 


## springboot基于关系型数据库实现

### spring-jdbc

springboot默认使用的数据源是<span class="stress">org.apache.tomcat.jdbc.pool.DataSource</span><br>
在<span class="stress">DataSourceConfiguration类中，使用@Bean注解，</span>创建了<span class="stress">org.apache.tomcat.jdbc.pool.DataSource</span>实例，参考代码如下：
		
	/**
	 * Tomcat Pool DataSource configuration.
	 */
	@ConditionalOnClass(org.apache.tomcat.jdbc.pool.DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.tomcat.jdbc.pool.DataSource", matchIfMissing = true)
	static class Tomcat extends DataSourceConfiguration {

		@Bean
		@ConfigurationProperties(prefix = "spring.datasource.tomcat")
		public org.apache.tomcat.jdbc.pool.DataSource dataSource(
				DataSourceProperties properties) {
			org.apache.tomcat.jdbc.pool.DataSource dataSource = createDataSource(
					properties, org.apache.tomcat.jdbc.pool.DataSource.class);
			DatabaseDriver databaseDriver = DatabaseDriver
					.fromJdbcUrl(properties.determineUrl());
			String validationQuery = databaseDriver.getValidationQuery();
			if (validationQuery != null) {
				dataSource.setTestOnBorrow(true);
				dataSource.setValidationQuery(validationQuery);
			}
			return dataSource;
		}

	}


<span class="stress">知识点：</span>
	
**1. @Bean 的用法**

@Bean注解是一个**方法级别上的注解**，主要用在@configuration注解的类里，也可以用在@Component注解的类里，**添加bean的id为方法名。**

**2. 例子**
	
下面是@Configuration里的一个例子
	
		@Configuration
		public class AppConfig {
		
		    @Bean
		    public TransferService transferService() {
		        return new TransferServiceImpl();
		    }
		
		}

等价于xml里的配置

		<beans>
	    	<bean id="transferService" class="com.acme.TransferServiceImpl"/>
		</beans>

**pom文件引用jar包**

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>


**配置文件**
	
	spring.datasource.url=jdbc:mysql://localhost:3306/springboot?useUnicode=true&ampcharacterEncoding=UTF-8&ampautoReconnect=true
							&ampfailOverReadOnly=false 
	spring.datasource.username=root
	spring.datasource.password=123456

更多配置信息参考<span class="stress">JdbcProperties</span>

**持久层实现**
	
	@Repository
	public class UserDaoImpl implements UserDao {
	
		@Autowired
		private JdbcTemplate template;
	
		
		/**
		 * 新增记录
		 */
		@Override
		public int addUser(User user) {
			String sql = "insert into user (name,create_time) values(?,?)";
			return template.update(sql, user.getName(),user.getCreateTime());
		}
	
		
		/**
		 * 根据id删除记录
		 */
		@Override
		public int delUserById(int id) {
			String sql = "delete from user where id=?";
			
			return template.update(sql, id);
		}
		
		/**
		 * 以实体类形式，返回单条记录
		 */
		@Override
		public User queryUser(User user) {
			String sql = "select * from user where id=?";
			return (User) template.query(sql, new MyRowMapper(User.class), user.getId()).get(0);
		}
	}


springboot自动在容器中创建JdbcTemplate实例。在<span class="stress">JdbcTemplateAutoConfiguration</span>类中创建的
代码如下：

	@Bean
	@Primary
	@ConditionalOnMissingBean(JdbcOperations.class)
	public JdbcTemplate jdbcTemplate() {
		JdbcTemplate jdbcTemplate = new JdbcTemplate(this.dataSource);
		JdbcProperties.Template template = this.properties.getTemplate();
		jdbcTemplate.setFetchSize(template.getFetchSize());
		jdbcTemplate.setMaxRows(template.getMaxRows());
		if (template.getQueryTimeout() != null) {
			jdbcTemplate
					.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
		}
		return jdbcTemplate;
	}


<span class="stress">java.sql.ResultSetMetaData</span>类详解

	An object that can be used to get information about the types and properties of the columns in a ResultSet object. 
	The following code fragment creates the ResultSet object rs, creates the ResultSetMetaData object rsmd, and uses rsmd to 
	find out how many columns rs has and whether the first column in rs can be used in a WHERE clause. 


     ResultSet rs = stmt.executeQuery("SELECT a, b, c FROM TABLE2");
     ResultSetMetaData rsmd = rs.getMetaData();
     int numberOfColumns = rsmd.getColumnCount();
     boolean b = rsmd.isSearchable(1);


ResultSetMetaData是一个借助ResultSet对象，获取字段的类型和属性信息


### spring-data-jpa

	
- **什么是jpa**

	jpa的全称是<span class="stress">java Persistence API（java持久化API）</span>，可以通过注解或xml来描述实体对象与关系表之间的映射关系。将实体对象持久化到数据库中。 

- **什么是spring-data-jpa**
	
	spring-data-jpa是spring提供的一套简化jpa开发的框架	，按照约定好的【方法命名规则】写dao层接口，就可以在不写接口实现的情况下操作数据库，同时提供了除了crud之外的很多功能。如分页，排序。
	<span class="stress">底层还是依赖Hibernate的JPA技术实现</span>

- **spring-data-jpa的好处？**
	
	开发人员只需操作实体对象，不用关注sql如何实现

- **springboot集成spring-data-jpa**

	1. 导入maven坐标
	
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-data-jpa</artifactId>
			</dependency>
	
	2. 配置文件配置jpa信息
	
			//表不存在，自动创建
			spring.jpa.hibernate.ddl-auto= update 
			//显示 sql 语句 
			spring.jpa.show-sql=true

	3. 基于jpa，实现对数据库访问demo

		- 配置文件配置参数

				spring.datasource.url=jdbc:mysql://localhost:3306/springboot?useUnicode=true&ampcharacterEncoding=UTF-8
				&ampautoReconnect=true&ampfailOverReadOnly=false 
				spring.datasource.username=root
				spring.datasource.password=123456
				
				//表不存在，自动创建
				spring.jpa.hibernate.ddl-auto= update 
				//显示 sql 语句 
				spring.jpa.show-sql=true
				
			**参考JpaProperties**

		- 创建实体类，使用jpa注解，实现实体类与关系表之间的映射

				@Entity(name="student")
				public class Student {
				
					@Id
					@GeneratedValue(strategy=GenerationType.AUTO)
					private int stuId;
					
					@Column(length=5,nullable=true)
					private String stuName;
					
					@Column(length=20)
					private String stuNo;
				
					public int getStuId() {
						return stuId;
					}
				
					//setter和getter方法省略

				}

		- 编写dao层接口，继承PagingAndSortingRepository

				public interface StudentRepository extends PagingAndSortingRepository<Student, Integer> {
					Student findStuByStuId(int id);
					List<Student> findStuByStuNameLike(String name);
					List<Student> findStuByStuIdBetween(int beginId,int endId);
				}
			基于注解查询语句

				@Query(value="select s from student s where s.stuNo=?1")
				Student findStuByStuNo(String stuNo);

			1、表名为@Entity(name="student")的name属性值<br>
			2、必须将所有字段全部查回来，否则无法转换成实体类<br>
			3、查询条件的字段为实体类的属性
			4、注解优先级高于方法命名规则

		- 测试

					@Autowired
					private StudentRepository repository;
					
					private LoggerUtil logger = LoggerUtil.getInstance(getClass());
					@Test
					public void insertStu(){
						Student entity = new Student("李打野", "2012080244");
						repository.save(entity);
					} 
					
					@Test
					public void qryStuById(){
						Student student = repository.findStuByStuId(1);
						logger.debug("id为1的学生新信息为{0}",student.toString());
					}
					
					@Test
					public void pageQry(){
						@SuppressWarnings("deprecation")
						Page<Student> page = repository.findAll(new PageRequest(0, 10,new Sort(Direction.DESC, "stuNo")));
						
						logger.debug("分页查询结果===={0}",page.getContent());
					}
					
					@Test
					public void findStuByStuNameLike(){
						List<Student> list = repository.findStuByStuNameLike("李%");
						
						logger.debug("根据姓名模糊查询的结果为{0}",list);
					}
					@Test
					public void findStuByStuIdBetween(){
						List<Student> list = repository.findStuByStuIdBetween(1,1);
						
						logger.debug("根据id范围查询的结果为{0}",list);
					}


		- spring-data-jpa dao层接口继承层级如图所示

			
			![](https://i.imgur.com/82FHwKe.png)

		- spring-data-jpa关键字
			
			<table class="tableOne" border="1" cellpadding="0" cellspacing="0">
			<!-- 标题 -->
			<caption><span class="stress">查询关键字</span></caption>
			<thead>
				<tr>
					<th>序号</th>
					<th>逻辑关键字</th>
					<th>关键字表达</th>
				</tr>
				
			</thead>
		<tbody>
			<tr>
				<td>1</td>
				<td> AND</td>
				<td> And</td>
			</tr>
			<tr>
				<td>2</td>
				<td >OR</td>
				<td >Or</td>
			</tr>
			<tr>
				<td>3</td>
				<td >AFTER</td>
				<td >After<font style="vertical-align: inherit;">， </font></font>IsAfter</td>
			</tr>
			<tr>
				<td>4</td>
				<td >BEFORE</td>
				<td >Before<font style="vertical-align: inherit;">， </font></font>IsBefore</td>
			</tr>
			<tr>
				<td>5</td>
				<td >CONTAINING</td>
				<td >Containing<font style="vertical-align: inherit;">，</font></font>IsContaining<font style="vertical-align: inherit;">，</font></font>Contains</td>
			</tr>
			<tr>
				<td>6</td>
				<td >BETWEEN</td>
				<td >Between<font style="vertical-align: inherit;">， </font></font>IsBetween</td>
			</tr>
			<tr>
				<td>7</td>
				<td >ENDING_WITH</td>
				<td >EndingWith<font style="vertical-align: inherit;">，</font></font>IsEndingWith<font style="vertical-align: inherit;">，</font></font>EndsWith</td>
			</tr>
			<tr>
				<td>8</td>
				<td >EXISTS</td>
				<td >Exists</td>
			</tr>
			<tr>
				<td>9</td>
				<td >FALSE</td>
				<td >False<font style="vertical-align: inherit;">，</font></font>IsFalse</td>
			</tr>
			<tr>
				<td>10</td>
				<td >GREATER_THAN</td>
				<td >GreaterThan<font style="vertical-align: inherit;">，</font></font>IsGreaterThan</td>
			</tr>
			<tr>
				<td>11</td>
				<td >GREATER_THAN_EQUALS</td>
				<td >GreaterThanEqual<font style="vertical-align: inherit;">，</font></font>IsGreaterThanEqual</td>
			</tr>
			<tr>
				<td>12</td>
				<td >IN</td>
				<td >In<font style="vertical-align: inherit;">，</font></font>IsIn</td>
			</tr>
			<tr>
				<td>13</td>
				<td >IS</td>
				<td >Is<font style="vertical-align: inherit;">，</font>Equals<font style="vertical-align: inherit;">(或没有关键字)</font></td>
			</tr>
			<tr>
				<td>14</td>
				<td >IS_NOT_NULL</td>
				<td >NotNull<font style="vertical-align: inherit;">，</font></font>IsNotNull</td>
			</tr>
			<tr>
				<td>15</td>
				<td >IS_NULL</td>
				<td >Null<font style="vertical-align: inherit;">，</font></font>IsNull</td>
			</tr>
			<tr>
				<td>16</td>
				<td >LESS_THAN</td>
				<td >LessThan<font style="vertical-align: inherit;">，</font></font>IsLessThan</td>
			</tr>
			<tr>
				<td>17</td>
				<td >LESS_THAN_EQUAL</td>
				<td >LessThanEqual<font style="vertical-align: inherit;">，</font></font>IsLessThanEqual</td>
			</tr>
			<tr>
				<td>18</td>
				<td >LIKE</td>
				<td >Like<font style="vertical-align: inherit;">，</font></font>IsLike</td>
			</tr>
			<tr>
				<td>19</td>
				<td >NEAR</td>
				<td >Near<font style="vertical-align: inherit;">，</font></font>IsNear</td>
			</tr>
			<tr>
				<td>20</td>
				<td >NOT</td>
				<td >Not<font style="vertical-align: inherit;">，</font></font>IsNot</td>
			</tr>
			<tr>
				<td>21</td>
				<td >NOT_IN</td>
				<td >NotIn<font style="vertical-align: inherit;">，</font></font>IsNotIn</td>
			</tr>
			<tr>
				<td>22</td>
				<td >NOT_LIKE</td>
				<td >NotLike<font style="vertical-align: inherit;">， </font></font>IsNotLike</td>
			</tr>
			<tr>
				<td>23</td>
				<td >REGEX</td>
				<td >Regex<font style="vertical-align: inherit;">，</font></font>MatchesRegex<font style="vertical-align: inherit;">，</font></font>Matches</td>
			</tr>
			<tr>
				<td>24</td>
				<td >STARTING_WITH</td>
				<td >StartingWith<font style="vertical-align: inherit;">，</font></font>IsStartingWith<font style="vertical-align: inherit;">，</font></font>StartsWith</td>
			</tr>
			<tr>
				<td>25</td>
				<td >TRUE</td>
				<td >True<font style="vertical-align: inherit;">，</font></font>IsTrue</td>
			</tr>
			<tr>
				<td>26</td>
				<td >WITHIN</td>
				<td >Within<font style="vertical-align: inherit;">，</font></font>IsWithin</td>
			</tr>
		</tbody>
		</table>


## springboot的事务处理

**一、事务有四个特性：ACID** 

**原子性**（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，
要么完全不起作用。 

**一致性**（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状
态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
 
**隔离性**（Isolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，
防止数据损坏。
 
**持久性**（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从
任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。 

**二、传播行为**

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运
行，也可能开启一个新事务，并在自己的事务中运行。 
 
<span class="stress">Spring 定义了七种传播行为：</span>
 
**PROPAGATION_REQUIRED** 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运
行。否则，会启动一个新的事务，Spring 默认使用 

**PROPAGATION_SUPPORTS** 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会
在这个事务中运行 

**PROPAGATION_MANDATORY** 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常 

**PROPAGATION_REQUIRED_NEW** 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果
存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用 JTATransactionManager 的话，则需要
访问 TransactionManager 

**PROPAGATION_NOT_SUPPORTED** 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期
间，当前事务将被挂起。如果使用 JTATransactionManager 的话，则需要访问 TransactionManager 

**PROPAGATION_NEVER** 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛
出异常 

**PROPAGATION_NESTED** 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务
可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED 一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的
文档来确认它们是否支持嵌套事务 
 
**三、隔离级别** 

隔离级别定义了一个事务可能受其他并发事务影响的程度。 
 
**ISOLATION_DEFAULT** 使用后端数据库默认的隔离级别，Spring 默认使用，mysql 默认的隔离级别为：
Repeatable Read(可重复读) 
 
**ISOLATION_READ_UNCOMMITTED** 读未提交，最低的隔离级别，允许读取尚未提交的数据变更，可能会导致
脏读、幻读或不可重复读 

**ISOLATION_READ_COMMITTED** 读已提交，允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读 或不可重复读仍有可能发生 

**ISOLATION_REPEATABLE_READ** 可重复读，对同一字段的多次读取结果都是一致的，除非数据是被本身事
务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生 

**ISOLATION_SERIALIZABLE** 可串行化，最高的隔离级别，完全服从 ACID 的隔离级别，确保阻止脏读、不可
重复读以及幻读，也是最慢的事务隔离级别，因为它通常是通过完全锁定事务相关的数据库表来实现的 

**脏读**（Dirty reads）——脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写再
稍后被回滚了，那么第一个事务获取的数据就是无效的。
 
**不可重复读**（Nonrepeatable read）——不可重复读发生在一个事务执行相同的查询两次或两次以上，但
是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。
 
**幻读**（Phantom read）——幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一
个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在
的记录。 
 
**四、属性说明 @Transactional** 

1. **isolation**：用于指定事务的隔离级别。默认为底层事务的隔离级别。 
2. **noRollbackFor**：指定遇到指定异常时强制不回滚事务。 
3. **noRollbackForClassName**：指定遇到指定多个异常时强制不回滚事务。该属性可以指定多个异常类名。 
4. **propagation**:指定事务的传播属性。 
5. **readOnly**：指定事务是否只读。表示这个事务只读取数据但不更新数据，这样可以帮助数据库引擎优化事务。若真的是一个只读取的数据库应设置 readOnly=true 
6. **rollbackFor**：指定遇到指定异常时强制回滚事务。 
7. **rollbackForClassName**：指定遇到指定多个异常时强制回滚事务。该属性可以指定多个异常类名。 
8. **timeout**：指定事务的超时时长


springboot实现事务，则只需在业务层的类的方法中加上@Transactional注解即可

代码实现：

	@Service
	public class StudentServiceImpl implements StudentService {
		
		@Autowired
		private StudentRepository repository;
		
		@Transactional
		@Override
		public void insert(Student entity) {
			repository.save(entity);
			
			//测试所用
			boolean flag = true;
			if(flag){
				throw new RuntimeException();
			}
			
		}
	
	}



## springboot集成h2数据库

- pom文件引用jar包

		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
		</dependency>
- 配置文件配置

		//#AUTO_SERVER:启动自动混合模式，允许开启多个连接，该参数不支持在内存中运行模式
		spring.datasource.url=jdbc:h2:~/test;AUTO_SERVER=TRUE;DB_CLOSE_ON_EXIT=FALSE 
		spring.datasource.username=sa 
		spring.datasource.password=

- 总结

	~：表示当前操作系统登录用户<br>
	/test：数据存储目录<br>
	自定义存储目录：file:d:/test
	
	h2是个嵌入式数据库，它本身是个类库，可以直接嵌入到应用项目中。纯java写的
	最大的用途是可以同应用程序一起打包发布，可以方便存储少量的结构化数据


## springboot集成redis

- **pom依赖**

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>

- **配置文件配置**(RedisProperties)
	
	1. **单机配置**
	
		springboot自己有一套默认的配置
	2. **集群配置**

			spring.redis.cluster.nodes=192.168.220.134:7000,192.168.220.134:7001,
			192.168.220.134:7002,192.168.220.134:7003,192.168.220.134:7004,192.168.220.134:7005
	
			spring.redis.cluster.max-redirects=8
	3. **哨兵配置**
			
			//哨兵节点
			spring.redis.sentinel.nodes=192.168.220.134:26379,192.168.220.134:26380
			spring.redis.sentinel.master=mymaster
	4. **公共配置**
	
			//redis连接池配置
			spring.redis.pool.maxIdle=300
			spring.redis.pool.minIdle=30
			spring.redis.pool.maxActive=300
			spring.redis.pool.maxWait=1000

- **RedisTemplate模板**

		/**
		 * jedis操作redis工具类
		 * @author sxq
		 * @time 2018年7月1日 下午4:47:47
		 *
		 */
		@Component
		public class RedisService {
		
			@Autowired
			private RedisTemplate redisTemplate;
			
			@SuppressWarnings("unchecked")
			public void set(final byte[] key,final byte[] value,final long liveTime){
				redisTemplate.execute(new RedisCallback<Object>() {
					@Override
					public Object doInRedis(RedisConnection connection)
							throws DataAccessException {
						
						connection.set(key, value);
						if(liveTime!=0){
							connection.expire(key, liveTime);
						}
						return 1L;
					}
					
				});
			}
			
			public void set(String key,String value){
				set(key, value,Consts.TIME_OUT);
			}
			
			public void set(String key,String value,long liveTime){
				set(key.getBytes(), value.getBytes(),Consts.TIME_OUT);
			}
			
			public String get(String key){
				return get(key.getBytes());
			}
			
			@SuppressWarnings("unchecked")
			public String get(final byte[] key){
				return (String) redisTemplate.execute(new RedisCallback<String>() {
		
					@Override
					public String doInRedis(RedisConnection connection)
							throws DataAccessException {
						if(connection.exists(key)){
							try {
								return new String(connection.get(key),"utf-8");
							} catch (UnsupportedEncodingException e) {
								e.printStackTrace();
							}
						};
						return null;
					}
					
				});
			}
		
			@SuppressWarnings("unchecked")
			public void del(String... key) {
				redisTemplate.execute(new RedisCallback<Object>() {
		
					@Override
					public Object doInRedis(RedisConnection connection)
							throws DataAccessException {
						for(int i=0;i<key.length;i++){
							connection.del(key[i].getBytes());
						}
						return 1L;
					}
					
				});
			}
			
			@SuppressWarnings("unchecked")
			public boolean exsit(String key){
				return (boolean) redisTemplate.execute(new RedisCallback<Object>() {
		
					@Override
					public Object doInRedis(RedisConnection connection)
							throws DataAccessException {
						return connection.exists(key.getBytes());
					}
					
				});
			}
		}

