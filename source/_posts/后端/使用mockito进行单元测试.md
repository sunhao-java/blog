title: 使用mockito进行单元测试
tags: [单元测试,mock]
date: 2019-10-10
categories: 后端
---

写这篇文档的原因：  
现在项目中的mock单元测试配置有一点问题，第一，单元测试很慢，测试类继承BaseTest，每次测试时都会去加载xml文件到spring容器中，而这个是没有必要，因为mock测试就是用于模拟 被测试类 所调用的其他类的方法，设定这些方法的返回值（即打桩），因此，不需要加载任何spring文件，除非你要调用真实方法操作数据库或者dubbo服务等接口。第二，代码中既使用了注解@mock,又使用了spring的配置方式，产生了冗余配置。基于上面问题，说明使用者对mockito的使用并不是很了解，所以在这边简单介绍一下。

<!-- more -->
# 使用注解自动生成mock类
在需要Mock的属性上标记@Mock注解或者@Spy注解，然后@RunWith(MockitoJUnitRunner.class)或者在setUp()方法中显示调用MockitoAnnotations.initMocks(this);生成Mock类即可。

> 关于@Mock和@Spy注解的区别：

使用@Mock注解的属性会被mock化，而使用@Spy的属性不会被mock化，即如果不打桩默认都会执行真实的方法，如果打桩则返回桩实现。  
（注：使用@Mock的属性如果想调用真实方法可以使用doCallRealMethod()方法或thenCallRealMethod()，使用@Spy的属性打桩是需要使用doReturn而不要使用thenReturn，使用thenReturn即使返回打桩实现，也会去掉真实方法，这应该是一个mockito的一个bug。)

# 自动注入Mock类到被测试类
只要在被测试类上标记@InjectMocks，Mockito就会自动将标记@Mock、@Spy等注解的属性值注入到被测试类中。  

# 完整例子：
  测试类
    
    public class ServiceTest{
    
    @Mock
    private Dao mockDao;
    
    //@Spy
    //private Dao spyDao = new Dao();
    
    @InjectMocks
    private Service service = new Service();
    
    @Before
    public void setup(){

    MockitoAnnotations.initMocks(this);
    }
    
    @Test
    public void mockCallRealMethod(){
    /*调用真实方法*/
    //  doCallRealMethod().when(mockDao).getName(Mockito.anyString());
    //  doCallRealMethod().when(mockDao).getSex(Mockito.anyString());
        when(mockDao.getName(Mockito.anyString())).thenCallRealMethod();
        when(mockDao.getSex(Mockito.anyString())).thenCallRealMethod();

        String[] userInfos = service.getUserInfos("1");
        System.out.println("name:"+userInfos[0]);
        System.out.println("sex:"+userInfos[1]);
    }
   
    @Test
    public void spyTest(){
    /*使用thenReturn方式打桩，依旧会调用真实方法*/
    //  when(spyDao.getName("1")).thenReturn("zhangsan");
    //  when(spyDao.getSex("1")).thenReturn("famle");

    /*使用doReturn方式打桩，则不会调用真实方法*/
    //  doReturn("zhangsan").when(spyDao).getName(Mockito.anyString());
    //  doReturn("famle").when(spyDao).getSex(Mockito.anyString());
        String[] userInfos = service.getUserInfos("1");
        System.out.println("name:"+userInfos[0]);
        System.out.println("sex:"+userInfos[1]);
    }
    }

被调用类，即被mock的类：
    public class Dao {
    
    public String getName(String id)
    {
        if(StringUtils.isEmpty(id)){
            return "no this people";
        }
        System.out.println("id:"+id+" --- name:dengzhi");
        return "dengzhi";
    }
    
    public String getSex(String id)
    {
        if(StringUtils.isEmpty(id)){
            return "no this people";
        }
        System.out.println("id:"+id+" --- sex:male");
        return "male";
    }
    }

被测试类：
    
    public class Service {
    Dao dao = new Dao();
    
    public String[] getUserInfos(String id)
    {
        String[] userInfos = new String[2];
        
        userInfos[0] = dao.getName(id);
        userInfos[1] = dao.getSex(id);
        return  userInfos;
    }
    }


# 总结
从上面的例子可以看出，使用@Mock和@Spy都属性是不再需要spring配置文件的注入的，如果要使用spring注入，则结合@Autowired使用。如下：

    正常的bean声明 	<bean id=”dao” class=”Dao”>
    
    mock 	<bean id="dao1"  class="org.mockito.Mockito" factory-method="mock">
            <constructor-arg value="DaoInterface"></constructor-arg>
            </bean>

             请注意到svc不变化，mock将自动注入进入。这是因为spring的bean容器，如果id一样，后声明的bean会覆盖前面的bean。
    
    spy 	<bean id="daoInst"  class="Dao"></bean>
            <bean id="dao2"  class="org.mockito.Mockito" factory-method="spy">
            <constructor-arg ref="daoInst"></constructor-arg>
            </bean>

             同样svc不变化，直接注入。请注意spy需要获得一个实例。

因此，建议项目中的mock测试去掉mock_service.xml配置文件，去掉继承BaseTest。这样做即去掉冗余配置，又加快测试速度。
