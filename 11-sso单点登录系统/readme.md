sso单点登陆系统：
#登陆：
##service层：
###业务逻辑：
从数据库取用户数据判断用户名和密码是否正确，
如果正确,java.util.UUID生成唯一随机的token,
token作为key,user对象作为value,set到redis,
并返回token;
###代码：
// 3、如果正确生成token。
String token = UUID.randomUUID().toString();
// 4、把用户信息写入redis，key：token value：用户信息
user.setPassword(null);
jedisClient.set("SESSION:" + token, JsonUtils.objectToJson(user));
// 5、设置Session的过期时间
jedisClient.expire("SESSION:" + token, SESSION_EXPIRE);
// 6、把token返回
return E3Result.ok(token);

##web层：
###业务逻辑：
调取service层，如果用户数据正确，service层
返回token，获取token，set到浏览器的cookie
###代码：
String token = e3Result.getData().toString();
//如果登录成功需要把token写入cookie
CookieUtils.setCookie(request, response, "login_token", token);