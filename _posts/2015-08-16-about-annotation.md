## 注解的分类 ##

1. 源码注解
   注解只在源码中存在，编译成.class时就不存在了

2. 编译时注解
   在源码和.class文件中都存在
   @Override @Deprecated @Suppvisewarnings

3. 运行时注解
   在运行阶段还起作用,甚至还会影响运行逻辑的注解
   eg. @Autowired  

## 自定义注解 ##

### 1. 语法要求 ###

	
	@Target({ElementType.METHOD,Element.TYPE})
	@Retention(RetentionPolicy.RUNTIME)
	@Inherited
	@Documented
    public @interface Description{
		//使用@interface关键字定义注解
		//成员类型是受限的，合法的类型包括原始类型及String,Class,Annotation,Enumeration
		//如果注解只有一个成员，则成员名必需取名为value(),在使用时可以忽略成员名和赋值号(=)
		String desc();//成员以无参无异常方式声明
		String author();
		int age() default 18;//成员不能有不确定的值,即要么有默认值,或者使用default为成员指定默认值,默认值不能为null
	}
    //注解的注解，即为元注解。
	//没有成员的注解,即为标识注解
	
 1. @Target 注解的作用域:

   + CONSTRUCTOR 构造方法声明
   + FIELD 字段声明
   + LOCAL_VARIABLE 局部变量声明
   + METHOD 方法声明
   + PAACKAGE 包声明
   + PARAMETER 参数声明
   + TYPE 类,接口声明
 
 2. @Retention 生命周期
  
   + SOURCE 只在源码显示,编译时会丢失
   + CLASS 编译时会记录到class中，运行时忽略
   + RUNTIME 运行时存在,可以通过反射读取

 3. @Inherited 允许子类继承
 
 4. @Documented 生成javadoc时会包含该注解
 
### 2. 解析注解 ###

 概念:通过反射获取类、函数或成员上的运行时注解信息，从而实现动态控制程序运行的逻辑
 
    public static void main(String[] args){
		//1.使用类加载器来加载类
		try{
			Class c = Class.forName("com.ann.test.CHild");
			//2.找到类上面的注解
			boolean isExist = c.isAnnotationPresent(Description.class);
			if(isExist){
				//3.拿到注解实例
				Description d = c.getAnnotation(Description.class);
				System.out.println(d.value());
			}
			//4.找到方法上的所有方法
			Method[] ms = c.getMethods();
			for(Method m:ms){
				boolean isMExist = m.isAnnotationPresent(Description.class);
				if(isMExist){
					Description d = (Description)m.getAnnotation(Description.class);
					System.out.println(d.value());
				}
			}
		    //另外一种解析方法
			for((Method m:ms){
				Annotation[] as = m.getAnnotation();
				for(Annotation a:as){
					if(a instanceof Description){
						Description d = (Description)a;
						System.out.println(d.value());
					}
				}
			}
		}catch(ClassNotFoundException e){
			
		}
	}
