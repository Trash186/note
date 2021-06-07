# Spring

## autowird

1、默认按照类型去容器查找

2、如果找到多个相同类型的组件，再将属性名作为组件的id去容器中查找

如果一个接口有两个实现类，只写autowired编译期间就会报错

例如：

```java
@Autowired
private MerChatService merchatService;
```

接口分别有两个实现类 ：

MerchantServiceImpl 、MerchantServiceImplTwo

如果想注入第二个，可以直接写成

```java
@Autowired
private MerChatService merchatServiceImplTwo;
```

byName:会自动在容器上下文中查找，和自己对象set方法后面的值对应的beanid，beanid首字母必须小写,需要保证bean的id唯一

byType:会自动在容器上下文中查找，和自己对象属性类型相同的bean，这种方式在配置文件中甚至可以省略beanid 直接写<bean class="com.dgut.pojo.dog">，需要包装bean的class唯一

放在set或者属性名称上，使用autowired也可以不写set方法

省去一个<property name="dog" ref="dog"></property>的语句

<bean id="dog" class="com.dgut.pojo.dog">





@Resource 与 @Autowired的区别：
相同点：都是用来自动装配的，都可以用在属性字段上

不同点：@Autowired 通过byType方式实现，而且必须要求这个对象存在

​				@Resource 默认通过byName方式实现，如果找不到名字，在通过byType实现，如果两个都找不到的情况下，就报错

执行顺序：@Autowired通过bytype的方式实现，@Resource通过byName 的方式实现

如果autowired不能唯一自动装配上属性，则需要通过@Qualifier(value="xxx")方式实现

