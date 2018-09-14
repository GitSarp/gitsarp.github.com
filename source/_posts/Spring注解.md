---
title: Spring注解
date: 2018-07-09 20:14:22
tags: Spring
categories: Spring
---
[spring中文文档](https://wizardforcel.gitbooks.io/spring-doc-3x/)  感谢！

## 注解
### 获取容器(上下文)
```
public static void main(String[] args) { 
    ApplicationContext ctx
        = new AnnotationConfigApplicationContext(AppConfig. class); 
    MyService myService = ctx.getBean(MyService.class); 
    myService.doStuff();
}
```
### 通用
- **@Scope**注解定义该bean的作用域范围，注解方式如@scope(“prototype”),配置方式<bean scope="xxx" name=...>.
	- singleton,容器内只有一个实例，或@Singleton
	- prototype,原型模式，每次通过容器的getBean方法获取prototype定义的Bean时，都将产生一个新的Bean实例
	- request,每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效，配置：
	```
	<web-app>
   ...
  <listener>
<listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
  </listener>
   ...
</web-app>
	```
	- session，配置同上，每次HTTP Session都会产生一个新的bean，同时该bean仅在当前HTTP session内有效
	- global session
- Bean注解主要用于方法上，有点类似于工厂方法

```
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```
和下面的配置类似
```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```
##### @AutoWired和@Resource(J2EE注解)
- @Autowired注解默认按照类型装配，如果容器中包含多个同一类型的Bean，那么启动容器时会报找不到指定类型bean的异常，解决办法是结合@Qualified注解进行限定，指定注入的bean名称。
- @Resource如果没有指定name属性，并且按照默认的名称仍然找不到依赖对象时， @Resource注解会回退到按类型装配。但一旦指定了name属性，就只能按名称装配了。
### 控制器注解@Controller
- **@RestController**,@RestController注解相当于@ResponseBody ＋ @Controller
  - 如果只是使用@RestController注解Controller，则Controller中的方法无法返回jsp页面，配置的视图解析器InternalResourceViewResolver不起作用，返回的内容就是Return 里的内容。
  - 如果需要返回到指定页面，则需要用 @Controller配合视图解析器InternalResourceViewResolver才行。
  - 如果需要返回JSON，XML或自定义mediaType内容到页面，则需要在对应的方法上加上@ResponseBody注解。

- @ModelAttribute
- @RequestMapping 既可以作用在**类级别**，也可以作用在方法级别。如果没有指定method，表示都可以接收。(还有变体@GetMapping和@PostMapping)
- **@RequestParam**用于方法入参，@RequestParam(value = "name", required = false) String name,required默认为true，必传
- **@PathVariable**,绑定url到参数上，支持正则
- 方法增加入参ModelMap modelMap，spring会创建实例，modelMap.put()可以用来向页面返回数据
```
@RequestMapping(value="/happy/{dayid}",method=RequestMethod.GET)
public String findPet(@PathVariable String dayid, Model mode) {
//使用@PathVariable注解绑定 {dayid} 到String dayid
}
```
- @RequestBody是指方法参数应该被绑定到HTTP请求Body上。
- @ResponseBody的作用是将返回类型直接输入到HTTP response body中，常用来输出JSON数据。

#### @Component与@Configuration
```
@Configuration
public static class Config {

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean());
    }
}
//@Component则需要这样写，否则就将有两个SimpleBean实例
@Component
public static class Config {
    @Autowired
    SimpleBean simpleBean;

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean);
    }
}
```
### 校验
- @Valid (在Controller响应方法的form对应实体入参上加入@Valid注解，在form对应实体的字段上添加@NotNull等注解)
```
@Null 限制只能为null 
@NotNull 限制必须不为null 
@AssertFalse 限制必须为false 
@AssertTrue 限制必须为true 
@DecimalMax(value) 限制必须为一个不大于指定值的数字 
@DecimalMin(value) 限制必须为一个不小于指定值的数字 
@Digits(integer,fraction) 限制必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction 
@Future 限制必须是一个将来的日期 
@Max(value) 限制必须为一个不大于指定值的数字 
@Min(value) 限制必须为一个不小于指定值的数字 
@Pattern(value) 限制必须符合指定的正则表达式 
@Size(max,min) 限制字符长度必须在min到max之间 
@Past 验证注解的元素值（日期类型）比当前时间早 
@NotEmpty 验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0） 
@NotBlank 验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的空格 
@Email 验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式 
```
eg.
```
	@ResponseBody
    @PostMapping(value="/add")
    public String add(@Valid Student student,BindingResult bindingResult){ 
		if(bindingResult.hasErrors()){
			.....
		}
```

```
	@Entity
	@Table(name="t_student")
	public class Student {
    @Id
    @GeneratedValue
    private Integer id;

    @NotEmpty(message="姓名不能为空！")
    @Column(length=50)
    private String name;

    @NotNull(message="年龄不能为空！")
    @Min(value=18,message="年龄必须大于18岁！")
    @Column(length=50)
    private Integer age;
```
### 其他注解
- @PostConstruct 和 @PreDestroy 方法 实现初始化和销毁bean之前进行的操作 
- lombok注解：可以不用生成get、set和构造方法，idea中安装lombok-plugin插件，另外勾选enable annotation processing
```
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
```
### 事务
- 开启事务注解<tx:annotation-driven transaction-manager="transactionManager" />,用@Transactional
```
//删除失败后回滚
@Service
public class CompanyServiceImpl implements CompanyService {
  @Autowired
  private CompanyDAO companyDAO;
  //设置传播、只读属性和回滚策略，回滚可定义其他异常，isolation定义隔离级别，timeout设置超时时间(s)
  @Transactional(propagation = Propagation.REQUIRED, readOnly = false, rollbackFor = Exception.class)
  public int deleteByName(String name) {

    int result = companyDAO.deleteByName(name);
    return company;
  }
  ...
}
```

- 传播级别

  | 事务传播行为类型          | 说明                                                         |
  | ------------------------- | ------------------------------------------------------------ |
  | PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 |
  | PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
  | PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常。               |
  | PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。                 |
  | PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |
  | PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常               |
  | PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类 似的操作 |

  ![transaction-propagation](/Users/freaxjj/Downloads/transaction-propagation.png)

- 隔离级别

  | 类型                | 说明                                               |
  | ------------------- | -------------------------------------------------- |
  | DEFAULT             | 采用数据库默认隔离级别                             |
  | READ_UNCOMMITTED    | 读未提交的数据（会出现脏读取）                     |
  | READ_COMMITTED      | 读已提交的数据（会出现幻读，即前后两次读的不一样） |
  | REPEATABLE_READ     | 可重复读，会出现幻读                               |
  | SERIALIZABLE 串行化 | （对资源消耗较大，一般不使用）                     |

   

