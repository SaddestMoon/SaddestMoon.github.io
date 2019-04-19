---
layout: post
title: 前后端分离项目集成CAS
description: "前后端分离项目集成CAS"
categories: 成长の足迹
tags: [笔记 , 代码狗 ,Java ,CAS集成]
imagefeature: 
comments: true
share: true
---


# 前后端分离的项目集成CAS #

这里说的前后端分离,是指部署也分离的项目.

关于前后端分离的项目如何集成CAS,很少有资料博客来说如何做.

本文先给出一个解决流程思路,再从原理层面来进行一定的剖析.

采用环境:`JDK1.8`,`Tomcat 8.5`,`cas 3.4.1`.

实际本文内容和`JDK`、`Tomcat`版本无关,理论上和`cas`版本也无关.

主要涉及以下知识:

1. `cas`认证原理,及客户端的认证源码分析.
2. `session`、`cookie`相关知识

主要解决如下两个问题:

1. 前后端分离项目如何接入`cas`.
2. 如何和项目原有的登录做集成.

## 问题一、前后端分离项目如何集成CAS ##

先说解决方案.

关键就是这两点:

1. 登录成功后跳转的第一个地址必须是被后端`CAS Filter`拦截的地址(主要用来重定向到前端,可以是一个接口,也可以是一个`.jsp`界面),保证`cas`可以把认证信息写入`session`.
2. 前端发起的后续请求必须和第一个请求(即前面的那个跳转后端的那个请求)是属于同一个`session`.

只要能保证这两点,那就绝对没问题.下面这是我的解决思路:

假设:

`cas`服务端地址为`192.168.0.90:8080/cas`

后端服务地址为`192.168.0.100:8080/api-server`

前端界面地址为`192.168.0.120`

后端`webapp`根目录下增加一个`cas.jsp`来做重定向,这个jsp的访问地址为`192.168.0.100:8080/api-server/cas.jsp`

具体步骤:

### 1. 修改`web.xml`增加对`cas.jsp`的拦截 ###

随手建一个`cas.jsp`,放到后端`webapp`的根目录下,里面暂时不用写什么代码,后面再加.

修改`web.xml`中的`CAS Filter`的配置,额外增加对`cas.jsp`的拦截.

    <filter-mapping>
		<filter-name>CAS Filter</filter-name>
		<url-pattern>/cas.jsp</url-pattern>
	</filter-mapping>

### 2.未登录时,前端重定向后端`cas.jsp` ###

调整下前端的js逻辑,在发现未登录时跳转`cas.jsp`(准确的说,最终目的是**保证登录成功后第一个跳转的界面是`cas.jsp`**).

    // 没有登录,跳转后端cas.jsp
    if(notLogin){
        window.location.href='192.168.0.100:8080/api-server/cas.jsp';
    }
    
`cas.jsp`被`CAS Filter`拦截,那么会再自动重定向到`cas`登录界面:`192.168.0.90:8080/cas/login?service=192.168.0.100:8080/api-server/cas.jsp`.

如果想节省一次重定向的开销,前端也可以直接重定向到登录界面,注意加上`service`参数即可

    //和上面代码结果一样,但是节省一次重定向开销.参考代码如下
    if(notLogin){
        window.location.href=`192.168.0.90:8080/cas/login?service=192.168.0.100:8080/api-server/cas.jsp`;
    }
    
如果一切顺利的话,这个时候登录成功,那么就会进入`cas.jsp`界面.这个进入`cas.jsp`的请求的`session`,是通过了`cas`认证的.

### 3. `cas.jsp`重定向到前端 ###

修改`cas.jsp`,加入如下代码:

    // 重定向到前端地址.
    String url = "192.168.0.120";
    response.sendRedirect(url);

那么此时,登录成功后最终会进入前端界面.下面我们只需要保证,后续前端发起的请求,和刚刚这个经过了`cas.jsp`的请求,是属于同一个`session`就行了,或者说,`JSEESIONID`这个`cookie`能正常写入前端的域下.

### 4. 保证将`JSESSIONID`写入前端`cookie`中 ###

在创建一个新的`session`后,`Tomcat`的`Session Manager`在`response`中会加上`Set-Cookie:JSESSIONID=123123`这个头,来让浏览器写入名为`JSESSIONID`的`cookie`.但是这个是写在后端的域下面而不是前端的.而一般情况下,`cookie`是没法跨域读写的.

后果就是,登录成功了也跳转回了前端界面,但是后续的请求因为请求中没有`JSESSIONID`这个`cookie`(实际上也不是必须放在`cookie`里),导致后续的每个请求,对于`Tomcat`来说,都是一个新的请求,,进而创建新的`session`--和第一个通过了认证的请求半毛钱关系都没,那`CAS Filter`自然也就不会放行.所以后续的请求依旧是调用不成功.

PS:如果这部分没看懂,那么建议还是看看后文中的CAS认证原理的章节.

那么这一步要解决的问题就是如何将`JSESSIONID`写入前端的域下的问题.

那大体上的方案,也就如下三种:

1. 允许跨域写`cookie`.大体上就是`ajax`请求加上`{crossDomain: true, xhrFields: {withCredentials: true}}`,后端响应头加上`response.addHeader("Access-Control-Allow-Credentials", "true")`.
2. 让前后端"不跨域".具体方法就是,用`nginx`将前后端反向代理到同一个域下,无论是访问前端界面还是调用后端接口亦或是后端`CAS Filter`中的配置**都是用这个代理后的地址**.
3. 前端手动写入`JSESSIONID`.比如,`cas.jsp`在重定向回前端时,在url后面附带上`JSESSIONID`的值,前端js获取到地址栏里的值,再手动写入`cookie`中.

就以第三种--让前端手动写入`JSESSIONID`为例:

修改`cas.jsp`.

    //获取sessionid
    String jsessionid = session.getId();
    // 前端地址.
    String url = "192.168.0.120";
    response.sendRedirect(url + "?jsessionid=" + jsessionid);

前端js:

    // getJsessionIdFromUrl() 从地址栏里获取jseesionid参数的值,具体逻辑自行实现
    var jsessionid = getJsessionIdFromUrl();
    // setCookie() 写入cookie,具体逻辑自行实现
    setCookie('jsessionid',jsessionid);

前端后续再发送的请求,带上`JSESSIONID`这个`cookie`,就没事了.

当然细节上还有很多可优化可完善的地方,但大致上就是这个流程思路.

### 方案流程 ###

上述方案的总体的流程就是:

1. 访问前端`192.168.0.120`
2. 前端判断没登录,跳转`192.168.0.100:8080/api-server/cas.jsp`
3. 访问`192.168.0.100:8080/api-server/cas.jsp`的请求到达`Tomcat`的`Session Manager`,发现没有`session`,创建新的`session`
4. 请求到达后端的`CAS Filter`,`CAS Filter`发现`session`未认证且没有`ST`参数,`302`让浏览器重定向到`cas`服务端登录界面`192.168.0.90:8080/cas/login?service=192.168.0.100:8080/api-server/cas.jsp`
5. 请求到到达`cas`服务端,服务端没有在自己的域(即`192.168.0.90:8080`)下找到名为`TGC`的`cookie`,那么认定是用户之前没有登录过,展示登录界面. 
6. 在`cas`服务端登录界面输入用户名密码,进行登录.
7. 登录成功,`cas`服务端根据`service`参数,`302`让浏览器重定向到`192.168.0.100:8080/api-server/cas.jsp?ticket=ST-ABC1234567`,注意此时是附带上了`ST`参数,并且在服务端的域(`192.168.0.90:8080`)下写入名为`TGC`的`cookie`.
8. 访问`192.168.0.100:8080/api-server/cas.jsp?ticket=ST-ABC1234567`到达`Tomcat`的`Session Manager`,发现没有`session`,创建新的`session`.
9. 请求进入后端`CAS Filter`,发现`session`未认证但是有`ST`参数,予以放行
10. 请求再进入后端的`CAS Validation Filter`,向`cas`服务端校验`ST`,校验通过,将校验结果声明(内含用户信息)写入此`session`.
11. 请求到达`192.168.0.100:8080/api-server/cas.jsp`界面
12. `cas.jsp`做一系列处理(比如二次登录)后,将`JSESSIONID`附带在url参数中,再重定向回前端
13. 前端从地址栏url中获取到`JSESSIONID`参数,写入`cookie`中.
14. 后续请求,带上`JSESSIONID`这个`cookie`.比如再去请求`192.168.0.100:8080/api-server/a/b/c-api`.
15. 请求到达`Tomcat`的`Session Manager`,发现有`JSESSIONID`,并且存在,不再创建新的`session`.
16. 请求到达后端`CAS Filter`,`CAS Filter`发现`session`中有认证信息.予以放行.
17. 请求到达`192.168.0.100:8080/api-server/a/b/c-api`


## 问题二、`CAS`如何和原有的登录认证做对接  ##

常见的一个问题就是,我这个项目本身已经有了一套登录体系,现在要集成`cas`,应当如何去做.

尤其是对接第三方的`cas`,`cas`服务端采用的用户库都和我们自己的不一样,这个该怎么去做.

大体思路依旧是通过前面我们增加的这个`cas.jsp`.在`cas.jsp`中根据`cas`返回的用户信息(一般都有用户名,但是不会有也不应当有密码),做二次登录,二次登录成功后再跳转前端.

参考代码如下:
    
    // cas.jsp 内容
    // cas会将用户信息写入session中. request.getAttribute("_const_cas_assertion_") 也可以拿到.
    Object object = request.getSession().getAttribute("_const_cas_assertion_");
    // org.jasig.cas.client.validation.Assertion
    Assertion assertion = (Assertion) object;
    // 获取到用户名
    String userName = assertion.getPrincipal().getName()
    /* 
    假设原有登录是调用的 loginService.login(userName,password);方法
    那我们增加一个免密登录方法 loginService.loginWithOutPWD(userName),
    里面的处理逻辑和返回值 同loginService.login(userName,password)基本一致,唯独不再需要密码.
     */
    Object obj = loginService.loginWithOutPWD(userName);
    // isLogin判断是否二次登录成功
    if(isLogin(obj)){
        // 二次登录成功,调整前端. 如果原有登录有其它参数需要给前端,也可以附带在url后面.
        //获取sessionid
        String jsessionid = session.getId();
        // 前端地址.
        String url = "192.168.0.120";
        response.sendRedirect(url + "?jsessionid=" + jsessionid);
    }else{
        // 二次登录失败
        //...
    }

## CAS认证原理 ##

### `CAS`基本概念 ###

#### 1.体系结构

从总体上看,CAS由两大部分组成:一个CAS Server 和多个CAS Client.

CAS Server(服务端)负责提供登录认证服务,单独部署,会给用户颁发两个核心票据:TGT(登录票据,服务端使用)和ST(服务票据,客户端使用).

CAS Client(客户端)负责处理对客户端受保护资源的访问请求.一般通过是在`web.xml`中配置了`CAS`过滤器,和应用系统部署在一起,可以有多个.

#### 2. 核心票据

CAS的核心就是其Ticket,及其在Ticket之上的一系列处理操作.CAS的主要票据有TGT、ST、PGT、PGTIOU、PT,其中TGT、ST是CAS1.0(基础模式)协议中就有的票据,PGT、PGTIOU、PT是CAS2.0(代理模式)协议中有的票据.这里主要介绍CAS1.0—基础模式中的几种票据.

##### TGT(Ticket Grangting Ticket)

TGT是CAS为用户签发的登录票据,拥有了`TGT`,用户就可以证明自己在CAS成功登录过.`TGT`封装了Cookie值以及此Cookie值对应的用户信息.用户在CAS认证成功后,生成一个`TGT`对象,放入自己的`session`;同时,CAS生成TGC(一个`cookie`,官方文档说是叫`TGC`,但是我这边看到的名称是`CASTGC`,可能是我理解问题或者是版本差异,下文统称`TGC`),写入浏览器.

`TGT`对象的id就是`TGC cookie`的值,当HTTP再次请求到来时,如果传过来的有`TGC`这个`cookie`,并且CAS能找到对应的`TGT`,则说明用户之前登录过;如果没有,则用户需要重新登录.

##### TGC(Ticket-granting cookie)

上面提到,CAS Server会生成`TGT`,而`TGC`就是将`TGT`的id以`cookie`形式放到浏览器端,是CAS Server用来明确用户身份的凭证.

##### ST(ServiceTicket)

ST是CAS为用户签发的访问某一服务票据.在登录成功后重定向回客户端的时候,会给客户端颁发ST,客户端的`CAS Validation Filter`会根据ST去服务端再次做校验,获取用户信息.

为了保证ST的安全性:ST 是基于随机生成的,没有规律性.而且,CAS规定 ST 只能存活一定的时间,然后 CAS Server 会让它失效.而且,CAS 协议规定ST只能使用一次,无论 Service Ticket 验证是否成功, CASServer 都会清除服务端缓存中的该 Ticket ,从而可以确保一个 Service Ticket 不被使用两次.

概括下就是,ST是服务端提供给客户端的用来获取用户信息的只能用一次的凭证.

#### 3. 核心过滤器

CAS Client使用的核心过滤器主要由两个:负责判断请求是否已认证的`AuthenticationFilter`和负责校验`ST`的`TicketValidationFilter`.

### `CAS`认证原理 ###

直接上官方提供的认证序列图(一定要看懂).

![cas认证序列图](https://i.imgur.com/U0WjEWZ.png)

其中的`ST`就是所谓的唯一令牌,用过即失效.这个令牌也可能是采用`JWT`来生成的.

如果`ST`采用的是`JWT`来进行生成,那么流程上稍微有些不同,序列图如下(这个了解就行):

![采用JWT的cas认证序列图](https://i.imgur.com/WOcrX1u.png)

可以打开Chrome的控制台或者是直接采用Filder进行抓包,对照着进行分析确认.

### CAS客户端核心过滤器 ###

对着源码来看下`cas`客户端的是如何判断一个`session`是否已通过认证.

前面我们提到过,CAS客户端使用的核心过滤器主要有两个:负责判断请求是否已认证的`AuthenticationFilter`和负责校验`ST`的`TicketValidationFilter`.

这两个过滤器都有多种不同的实现,下文只以具体的某个实现类为例来说明.

#### AuthenticationFilter

这个过滤器就是用来负责校验每个请求是否是认证的请求,如果请求未通过认证且不包含`ST`,那么就重定向到服务端登录界面.

对应客户端`web.xml`中配置的`CAS Filter`,大致是这样的:

    <filter>
        <filter-name>CAS Filter</filter-name>
    	<filter-class>org.jasig.cas.client.authentication.AuthenticationFilter</filter-class>
    	<init-param>
			<param-name>casServerLoginUrl</param-name>
			<param-value>${cas.serverUrl}/login</param-value>
		</init-param>
		<init-param>
			<param-name>serverName</param-name>
			<param-value>${cas.clientUrl}</param-value>
		</init-param>
		<init-param>
			<param-name>ignorePattern</param-name>
			<param-value>.*/login|.*/unsafe|.*/api/app/token/*|.*\.ico|.*\.js(?!p)|.*\.css|</param-value>
		</init-param>
		<init-param>
			<param-name>ignoreUrlPatternType</param-name>
			<param-value>REGEX</param-value>
		</init-param>
		...
    </filter>
    <filter-mapping>
		<filter-name>CAS Filter</filter-name>
		<url-pattern>/cas.jsp</url-pattern>
	</filter-mapping>
	<filter-mapping>
		<filter-name>CAS Filter</filter-name>
		<url-pattern>/api/*</url-pattern>
	</filter-mapping>

看下对应的实现.

    public final void doFilter(final ServletRequest servletRequest, final ServletResponse servletResponse,
            final FilterChain filterChain) throws IOException, ServletException {
        
        final HttpServletRequest request = (HttpServletRequest) servletRequest;
        final HttpServletResponse response = (HttpServletResponse) servletResponse;
        
        // 判断请求是否不需要过滤
        if (isRequestUrlExcluded(request)) {
            logger.debug("Request is ignored.");
            filterChain.doFilter(request, response);
            return;
        }
        
        final HttpSession session = request.getSession(false);
        // CONST_CAS_ASSERTION = "_const_cas_assertion_"
        final Assertion assertion = session != null ? (Assertion) session.getAttribute(CONST_CAS_ASSERTION) : null;
        // 存在assertion,即认为这是一个已通过认证的请求.予以放行
        if (assertion != null) {
            filterChain.doFilter(request, response);
            return;
        }
        // 不存在 assertion,那么就来判断这个请求是否是用来校验ST的(校验通过后会将信息写入assertion)
        final String serviceUrl = constructServiceUrl(request, response);
        final String ticket = retrieveTicketFromRequest(request);
        final boolean wasGatewayed = this.gateway && this.gatewayStorage.hasGatewayedAlready(request, serviceUrl);
        // 是校验ST的请求,予以放行
        if (CommonUtils.isNotBlank(ticket) || wasGatewayed) {
            filterChain.doFilter(request, response);
            return;
        }

        final String modifiedServiceUrl;

        logger.debug("no ticket and no assertion found");
        if (this.gateway) {
            logger.debug("setting gateway attribute in session");
            modifiedServiceUrl = this.gatewayStorage.storeGatewayInformation(request, serviceUrl);
        } else {
            modifiedServiceUrl = serviceUrl;
        }

        logger.debug("Constructed service url: {}", modifiedServiceUrl);

        // 要重定向界面地址(cas服务端登录界面).
        final String urlToRedirectTo = CommonUtils.constructRedirectUrl(this.casServerLoginUrl,
                getProtocol().getServiceParameterName(), modifiedServiceUrl, this.renew, this.gateway);

        logger.debug("redirecting to \"{}\"", urlToRedirectTo);
        this.authenticationRedirectStrategy.redirect(request, response, urlToRedirectTo);
    }
    
可以看到,`cas`正是通过`session`中是否有`assertion`的信息来判断一个请求是否合法.
    
而这个`assertion`信息,又是在客户端校验`ST`之后(登录成功后重定向回客户端的请求附带有ST参数)写入`session`中的.具体见下文`TicketValidationFilter`

#### TicketValidationFilter

这个过滤器用来校验`ST`,并在`session`中写入认证声明信息`assertion`.

对应客户端`web.xml`中配置的`CAS Validation Filter`,大致是这样的:
    
    <filter>
		<filter-name>CAS Validation Filter</filter-name>
		<filter-class>
			org.jasig.cas.client.validation.Cas20ProxyReceivingTicketValidationFilter
		</filter-class>
	</filter>
	
查看`Cas20ProxyReceivingTicketValidationFilter`源码,可以在其父类`AbstractTicketValidationFilter`的`doFilter()`方法中找到对应的代码,如下:
	
    final Assertion assertion = this.ticketValidator.validate(ticket,
                        constructServiceUrl(request, response));

    logger.debug("Successfully authenticated user: {}", assertion.getPrincipal().getName());

    // CONST_CAS_ASSERTION = "_const_cas_assertion_"
    request.setAttribute(CONST_CAS_ASSERTION, assertion);

    // useSession 对应配置项中的useSession参数,缺省值为true.但这个配置在3.4版本之后是弃用的,后续随时可能会被移除.
    if (this.useSession) {
        request.getSession().setAttribute(CONST_CAS_ASSERTION, assertion);
    }
    
那么通过这两个过滤器的源码分析,可以证明这两点:

1. cas客户端是根据`session`中是否有`assertion`信息来判断是否已认证.
2. `assertion`是在登录成功后重定向回后端的时候写入`session`的.

那么进而就会得出我们前文中提到的解决思路的关键性两点:

1. 登录成功后跳转的第一个地址必须是被后端`CAS Filter`拦截的地址(主要用来重定向到前端,可以是一个接口,也可以是一个`.jsp`界面),保证`cas`可以把认证声明信息`assertion`写入`session`.
2. 前端发起的后续请求必须和第一个请求(即前面的那个跳转后端的那个请求)是属于同一个`session`(因为有`assertion`,一定可以通过认证).

关于第一点不再多说.

关于第二点,如何保证前端发起的后续请求和第一个请求是同一个`session`,答案就是保证`JSEESIONID`一致就行.

### 关于`session`、`cookie`及`JSESSIONID`在其中起到的作用 ###

众所周知,Http协议是一种无状态协议,每次服务端接收到客户端的请求时,都是一个全新的请求,服务器并不知道客户端的历史请求记录；

为了弥补Http的无状态特性,`session`应运而生.服务器可以利用`session`存储客户端在同一个会话期间的一些操作记录,而服务端的这个`session`,对应到浏览器端,则是名为`JSESSIONID`的`cookie`,`JSESSIONID`的值就是`session`的id.

那么再看这两个问题:

1. 服务器如何判断客户端发送过来的请求是属于同一个`seesion`?

答:用`session`的id来进行区分,如果id相同,那就认为是同一个会话.在`Tomcat`中,`session`的id的默认名字是`JSESSIONID`.对应到前端就是名为`JSESSIONID`的`cookie`.

2. `session`的id是在什么时候创建,又是怎样在前后端传输的?

答:`tomcat`在**第一次**接收到一个请求时,会创建一个`session`对象,同时生成一个`session id`,并通过响应头的`Set-Cookie:"JSESSIONID=XXXXXXX"`命令,向客户端发送要求设置`Cookie`的响应.

前端在后续的每次请求时,会带上所有`cookie`信息,自然也就包含了`JSESSIONID`这个`cookie`.`Tomcat`据此来查找到对应的`session`.如果指定`session`不存在(比如我们随手编一个`JSESSIONID`,那对应的`session`肯定不存在),那么就会创建一个新的`session`,其id的值就是请求中的`JSESSIONID`的值.

这样我们在代码中就可以通过`request.getSession()`来获取到当前的会话信息.

PS:
实际上,`JSESSIONID`传给后端的方式,还可以直接在url后面加上`;jsessionid=xxxx`来传递.但是会被`cookie`中的`JSESSIONID`覆盖.
    
那么具体到cas对接上,第一次重定向回客户端的请求肯定是可以通过`cas`的认证的,那么只要后续的请求和第一个是同一个`session`,那就一定也能通过`cas`的过滤.

前面我们也说了,只要请求中的`JSESSIONID`是一致的,那就会被认定是同一个`session`.

前文中给出的解决思路,也正是基于此进行.

原理部分基本上就到这里.

如果觉得看懂了原理,那我最后提个问题:

如果通过nginx将前后端反向代理到同一个域下,登录成功后直接跳转前端地址,那是否可行?原因是什么?

以上.

参考文章:

[cas官方文档](https://apereo.github.io/cas/6.0.x/) https://apereo.github.io/cas/6.0.x/

[cas认证原理](https://blog.csdn.net/wang379275614/article/details/46337529) https://blog.csdn.net/wang379275614/article/details/46337529
