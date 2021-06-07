# Shiro

![](C:\Users\Lenovo\Desktop\1619405263(1).png)

Subject: 用户主体（把操作交给securityManager）

SecurityManager:安全管理器 （管理realm）

realm：shiro 连接数据的桥梁



## ShiroConfig

```java
@Configuration
public class ShiroConfig {

    /**
     * 创建ShiroFilterFactoryBean
     */
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager")DefaultWebSecurityManager securityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();

        //设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(securityManager);

        //添加Shiro内置过滤器
        /**
         * Shiro内置过滤器，可以实现权限相关的拦截器
         *    常用的过滤器：
         *       anon: 无需认证（登录）可以访问
         *       authc: 必须认证才可以访问
         *       user: 如果使用rememberMe的功能可以直接访问
         *       perms： 该资源必须得到资源权限才可以访问
         *       role: 该资源必须得到角色权限才可以访问
         */
        Map<String,String> filterMap = new LinkedHashMap<String,String>();

//        filterMap.put("/add","authc");
//        filterMap.put("/update","authc");
        //对于上述情况，可采用通配的方式
        filterMap.put("/*","authc");

       filterMap.put("/testThymeleaf", "anon");//放行，无需登录验证
//        //放行login.html页面
//        filterMap.put("/login", "anon");
//
//        //授权过滤器
//        //注意：当前授权拦截后，shiro会自动跳转到未授权页面
         filterMap.put("/add", "perms[user:add]");
//        filterMap.put("/update", "perms[user:update]");
//
//        filterMap.put("/*", "authc");

        //修改调整的登录页面
        shiroFilterFactoryBean.setLoginUrl("/toLogin");
        //设置未授权提示页面
        shiroFilterFactoryBean.setUnauthorizedUrl("/noAuth");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);


        return shiroFilterFactoryBean;
    }

    /**
     * 创建DefaultWebSecurityManager
     */
    @Bean(name="securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm")UserRealm userRealm){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联realm
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    /**
     * 创建Realm
     */
    @Bean(name="userRealm")
    public UserRealm getRealm(){
        return new UserRealm();
    }

//    /**
//     * 配置ShiroDialect，用于thymeleaf和shiro标签配合使用
//     */
//    @Bean
//    public ShiroDialect getShiroDialect(){
//        return new ShiroDialect();
//    }
}

```

## UserRealm

```java
public class UserRealm extends AuthorizingRealm
{
    /**
     * 执行授权
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection)
    {
        System.out.println("执行授权");
        SimpleAuthorizationInfo info=new SimpleAuthorizationInfo();
        info.addStringPermission("user:add");
        return info;
    }

    /**
     * 执行认证
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException
    {
        String name="xiao";
        String password="123456";

        UsernamePasswordToken token=(UsernamePasswordToken)authenticationToken;
        if(!token.getUsername().equals(name))
        {
            //用户名不存在
            return null;
        }

        return new SimpleAuthenticationInfo("",password,"");
    }
}

```

## DemoController

```java
@Controller
public class DemoController
{
    @RequestMapping("/testThymeleaf")
    public String testThymeleaf(Model model)
    {
        model.addAttribute("name","xiao");

        return "test";
    }
    @RequestMapping("/add")
    public String add(){
        return "/user/add";
    }

    @RequestMapping("/update")
    public String update(){
        return "/user/update";
    }

    @RequestMapping("/toLogin")
    public String toLogin(){
        return "/login";
    }

    @RequestMapping("/login")
    public String login(String name,String password,Model model)
    {
        /**
         * shiro编写认证操作
         */
        Subject subject = SecurityUtils.getSubject();

        //封装用户数据
        UsernamePasswordToken token = new UsernamePasswordToken(name, password);

        //执行登录方法
        try
        {
            subject.login(token);

            return "redirect:/testThymeleaf";
        }
        catch (UnknownAccountException e)
        {
            model.addAttribute("msg", "用户名不存在");
            return "login";
        }
        catch (IncorrectCredentialsException e)
        {
            model.addAttribute("msg", "密码错误");
            return "login";
        }

    }
}

```

