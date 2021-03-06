# Autofac Annotation Configuration [Autofac标签配置]

支持netcore2.0 + framework4.6+

## 如何使用
### NUGET Install-Package Autofac.Annotation
```
var builder = new ContainerBuilder();

// 注册autofac打标签模式
builder.RegisterModule(new AutofacAnnotationModule(typeof(AnotationTest).Assembly));
//如果需要开启支持循环注入
//builder.RegisterModule(new AutofacAnnotationModule(typeof(AnotationTest).Assembly).SetAllowCircularDependencies(true));
var container = builder.Build();
var serviceB = container.Resolve<B>();

```

AutofacAnnotationModule有两种构造方法
1. 可以传一个Assebly列表 （这种方式会注册传入的Assebly里面打了标签的类）
2. 可以传一个AsseblyName列表 (这种方式是先会根据AsseblyName查找Assebly 然后在注册)

## Supported Attributes [支持的标签说明]
### Bean标签
说明：只能打在class上面 把某个类注册到autofac容器
例如：
1. 无构造方法的方式	等同于 builder.RegisterType<A>();
```
//把class A 注册到容器
[Bean]
public class A
{
	public string Name { get; set; }
}
```
2. 指定Scope [需要指定AutofacScope属性 如果不指定为则默认为AutofacScope.InstancePerDependency]
```
    [Bean(AutofacScope = AutofacScope.SingleInstance)]
    public class A
    {
        public string Name { get; set; }
    }
```
3. 指定类型注册 等同于 builder.RegisterType<A6>().As<B>()
```
    public class B
    {

    }
	
    [Bean(typeof(B))]
    public class A6:B
    {

    }
```
4. 指定名字注册 等同于 builder.RegisterType<A6>().Keyed<A4>("a4")
```
    [Bean("a4")]
    public class A4
    {
        public string School { get; set; } = "测试2";
    }
```
5. 其他属性说明
* InjectProperties 是否默认装配属性 【默认为true】
* InjectPropertyType 属性自动装配的类型
	1. Autowired 【默认值】代表打了Autowired标签的才会自动装配
	2. ALL 代表会装配所有 等同于 builder.RegisterType<A>().PropertiesAutowired()
* AutoActivate 【默认为false】 如果为true代表autofac build完成后会自动创建 具体请参考 [autofac官方文档](https://autofaccn.readthedocs.io/en/latest/configuration/xml.html)
* Ownership 【默认为空】 具体请参考 [autofac官方文档](https://autofaccn.readthedocs.io/en/latest/configuration/xml.html)
* Interceptor 【默认为空】指定拦截器的Type
* InterceptorType 拦截器类型 拦截器必须实现 Castle.DynamicProxy的 IInterceptor 接口， 有以下两种
	1. Interface 【默认值】代表是接口型 
	2. Class 代表是class类型   这种的话是需要将要拦截的方法标virtual
* InterceptorKey 如果同一个类型的拦截器有多个 可以指定Key
* InitMethod 当实例被创建后执行的方法名称 类似Spring的init-method
	可以是有参数(只能1个参数类型是IComponentContext)和无参数的方法
* DestroyMetnod 当实例被Release时执行的方法 类似Spring的destroy-method
	必须是无参数的方法
```
    [Bean(InitMethod = "start",DestroyMetnod = "destroy")]
    public class A30
    {
        [Value("aaaaa")]
        public string Test { get; set; }

        public A29 a29;

        void start(IComponentContext context)
        {
            this.Test = "bbbb";
            a29 = context.Resolve<A29>();
        }

        void destroy()
        {
            this.Test = null;
            a29.Test = null;
        }
    }
	
```	

```
    public class B
    {

    }
	
    [Bean(typeof(B),"a5")]
    public class A5:B
    {
        public string School { get; set; } = "测试a5";
        public override string GetSchool()
        {
            return this.School;
        }
    }
```

### Autowired 自动装配
可以打在Field Property 构造方法的Parameter上面
其中Field 和 Property 支持在父类
```
    [Bean]
    public class A16
    {
	public A16([Autowired]A21 a21)
        {
            Name = name;
            A21 = a21;
        }
		
        [Autowired("A13")]
        public B b1;


        [Autowired]
        public B B { get; set; }
		
	//Required默认为true 如果装载错误会抛异常出来。如果指定为false则不抛异常
	[Autowired("adadada",Required = false)]
        public B b1;
    }
```

### Value 和 PropertySource
* PropertySource类似Spring里面的PropertySource 可以指定数据源
支持 xml json格式 支持内嵌资源

1. json格式的文件
```/file/appsettings1.json
{
  "a10": "aaaaaaaaa1",
  "list": [ 1, 2, 3 ],
  "dic": {
    "name": "name1"
  },
  "testInitField": 1,
  "testInitProperty": 1,
}
```
```
    [Bean]
    [PropertySource("/file/appsettings1.json")]
    public class A10
    {
        public A10([Value("#{a10}")]string school,[Value("#{list}")]List<int> list,[Value("#{dic}")]Dictionary<string,string> dic)
        {
            this.School = school;
            this.list = list;
            this.dic = dic;

        }
        public string School { get; set; }
        public List<int> list { get; set; } 
        public Dictionary<string,string> dic { get; set; } 
		
	[Value("#{testInitField}")]
        public int test;
		
	[Value("#{testInitProperty}")]
        public int test2 { get; set; }
		
	//可以直接指定值
	[Value("2")]
	public int test3 { get; set; }
    }
```

2. xml格式的文件
```appsettings1.xml
<?xml version="1.0" encoding="utf-8" ?>
<autofac>
  <a11>aaaaaaaaa1</a11>
  <list name="0">1</list>
  <list name="1">2</list>
  <list name="2">3</list>
  <dic name="name">name1</dic>
</autofac>

```

```
    [Bean]
    [PropertySource("/file/appsettings1.xml")]
    public class A11
    {
        public A11([Value("#{a11}")]string school,[Value("#{list}")]List<int> list,[Value("#{dic}")]Dictionary<string,string> dic)
        {
            this.School = school;
            this.list = list;
            this.dic = dic;

        }
        public string School { get; set; }
        public List<int> list { get; set; } 
        public Dictionary<string,string> dic { get; set; } 
    }
```

3. 不指定PropertySource的话会默认从工程目录的 appsettings.json获取值
