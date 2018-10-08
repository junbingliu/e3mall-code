#购物车工程业务逻辑
根据用户是否登陆决定购物车的商品是从cookie中还是redis还是cookie和redis结合中存取。
#web层：
##逻辑：
spring容器配置拦截器，拦截所有的请求，
##代码：
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="cn.e3mall.cart.interceptor.LoginInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
##逻辑：
实现org.springframework.web.servlet.HandlerInterceptor接口，
重写preHandle方法
从请求中取cookie，cookie中取token，
如果能取到token，调取sso单点登陆系统的service层接口方法，用token从redis获取user对象（token作为redis的key），
    E3Result e3Result = tokenService.getUserByToken(token);
1.如果取不到user对象证明没有登陆直接放行；
2.如果能取到user对象证明已登陆，把user对象set到request后再放行。
##代码：
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    // 前处理，执行handler之前执行此方法。
    //返回true，放行	false：拦截
    //1.从cookie中取token
    String token = CookieUtils.getCookieValue(request, "token");
    //2.如果没有token，未登录状态，直接放行
    if (StringUtils.isBlank(token)) {
        return true;
    }
    //3.取到token，需要调用sso系统的服务，根据token取用户信息
    E3Result e3Result = tokenService.getUserByToken(token);
    //4.没有取到用户信息。登录过期，直接放行。
    if (e3Result.getStatus() != 200) {
        return true;
    }
    //5.取到用户信息。登录状态。
    TbUser user = (TbUser) e3Result.getData();
    //6.把用户信息放到request中。只需要在Controller中判断request中是否包含user信息。放行
    request.setAttribute("user", user);
    return true;
}

#service层：
##逻辑：
从request中是否能获取user对象判断是否已登陆。
##代码：
//判断用户是否为登录状态
User user = (User) request.getAttribute("user");