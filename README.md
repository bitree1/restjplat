restjplat
=========

一个包含web服务异常处理和普通异常处理的小框架

业务场景
当一个项目中包含有web服务（rest）时，当遇见异常时，需要返回给客户端json格式的数据；但是项目中非web服务的异常则需要正常处理的情况。即同一个web项目中包含两种不同的异常处理。</p>

简单说明
直接使用Spring MVC架构web服务 ，目前只支持json返回值
1.所有产生异常先经过restservice-servlet.xml中注册的rest服务异常类处理


&lt;bean id=&quot;RestServiceExceptionHandler&quot;
		class=&quot;com.restjplat.quickweb.web.restServer.AppExceptionHandler&quot;&gt;
		&lt;property name=&quot;order&quot; value=&quot;1&quot; /&gt;
		&lt;property name=&quot;errorResolver&quot;&gt;
			&lt;bean class=&quot;com.restjplat.core.webserver.DefaultRestErrorResolver&quot;&gt;
				&lt;property name=&quot;exceptionMappingDefinitions&quot;&gt;
					&lt;map&gt;
						&lt;entry key=&quot;com.restjplat.core.webserver.RestServiceException&quot;
							value=&quot;404&quot; /&gt;
						&lt;entry key=&quot;Throwable&quot; value=&quot;500&quot; /&gt;
					&lt;/map&gt;
				&lt;/property&gt;
			&lt;/bean&gt;
		&lt;/property&gt;
	&lt;/bean&gt;


2.在 AppExceptionHandler 类中判断异常是否为rest异常，如果不是，则交给下一级异常处理。
	
//resources.properties 文件内定义了 凡是以 /rv1开头的请求就是rest服务
@Override
public boolean isRestServiceException(HttpServletRequest request){
	if(request.getServletPath().startsWith(Sysutils.getWebServerindex())) return true;
	else return false;
}


3.非rest异常则直接由spring处理
	<!-- 将Controller抛出的异常转到特定View,保持SiteMesh的装饰效果 ,
	并且该resolver的order大于rest服务error的梳理顺序 ,即所有的异常先过一遍rest-->
&lt;bean
		class=&quot;org.springframework.web.servlet.handler.SimpleMappingExceptionResolver&quot;&gt;
		&lt;property name=&quot;exceptionMappings&quot;&gt;
			&lt;props&gt;
				&lt;prop key=&quot;java.lang.Throwable&quot;&gt;error/500&lt;/prop&gt;
			&lt;/props&gt;
		&lt;/property&gt;
	&lt;/bean&gt;


