---
date: 2016-02-01 18:39
status: public
title: Java动态代理
---

###代理：设计模式
代理是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息，过滤消息并转发消息，以及进行消息被委托类执行后的后续处理。
如何理解？通俗的将就是方法增强。
###Java的动态代理
Java 动态代理机制以巧妙的方式近乎完美地实践了代理模式的设计理念。
Java动态代理是指的对Interface接口的代理，仅支持interface代理。这也是Java动态代理的缺点。
###Java代理机制相关的类和接口
学习Java动态代理，首先需要了解以下相关的接口
- java.lang.reflect.Proxy：这是 Java 动态代理机制的主类，它提供了一组静态方法来为一组接口动态地生成代理类及其对象。
Proxy类的相关法发：
```
// 方法 1: 该方法用于获取指定代理对象所关联的调用处理器
static InvocationHandler getInvocationHandler(Object proxy)  
// 方法 2：该方法用于获取关联于指定类装载器和一组接口的动态代理类的类对象
static Class getProxyClass(ClassLoader loader, Class[] interfaces)  
// 方法 3：该方法用于判断指定类对象是否是一个动态代理类
static boolean isProxyClass(Class cl)  
// 方法 4：该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例
static Object newProxyInstance(ClassLoader loader, Class[] interfaces, 
    InvocationHandler h)
```
- java.lang.reflect.InvocationHandler：这是调用处理器接口，它自定义了一个 invoke 方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问。
InvocationHandler调用处理器接口的方法
```
// 该方法负责集中处理动态代理类上的所有方法调用。第一个参数既是代理类实例，第二个参数是被调用的方法对象
// 第三个方法是调用参数。调用处理器根据这三个参数进行预处理或分派到委托类实例上发射执行
Object invoke(Object proxy, Method method, Object[] args)
```
- java.lang.ClassLoader：这是类装载器类，负责将类的字节码装载到 Java 虚拟机（JVM）中并为其定义类对象，然后该类才能被使用。Proxy 静态方法生成动态代理类同样需要通过类装载器来进行装载才能使用，它与普通类的唯一区别就是其字节码是由 JVM 在运行时动态生成的而非预存在于任何一个 .class 文件中。每次生成动态代理类对象时都需要指定一个类装载器对象.
###Java动态代理的使用
1. 通过实现 InvocationHandler 接口创建自己的调用处理器
2. 使用Proxy 的静态方法 newProxyInstance创建代理
```
public class InvocationHandlerImpl implements java.lang.reflect.InvocationHandler {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //处理自己的逻辑
        return null;
    }
}
```

```
// InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
InvocationHandler handler = new InvocationHandlerImpl(..);  
// 通过 Proxy 直接创建动态代理类实例
Interface proxy = (Interface)Proxy.newProxyInstance( classLoader, 
     new Class[] { Interface.class }, 
     handler );
 ```
 ###通过代码来理解
 Main类中实现一个Button，Button有两个方法，一个setOnClickListener和onClick，当调用Button的onClick时，触发的事件是Main类中的click方法
 Button 类
```
public class Button
{
	private OnClickListener listener;
	public void setOnClickLisntener(OnClickListener listener)
	{

		this.listener = listener;
	}

	public void click()
	{
		if (listener != null)
		{
			listener.onClick();
		}
	}
}
```
OnClickListener接口
```
public interface OnClickListener
{
	void onClick();
}
```
OnClickListenerHandler实现 InvocationHandler接口
```
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map; 
public class OnClickListenerHandler implements InvocationHandler
{
	private Object targetObject; 
	public OnClickListenerHandler(Object object)
	{
		this.targetObject = object;
	} 
	private Map<String, Method> methods = new HashMap<String, Method>(); 
	public void addMethod(String methodName, Method method)
	{
		methods.put(methodName, method);
	} 
	@Override
	public Object invoke(Object proxy, Method method, Object[] args)
			throws Throwable
	{ 
		String methodName = method.getName();
		Method realMethod = methods.get(methodName);
		return realMethod.invoke(targetObject, args);
	} 
}
```
在Main中调用
```
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
 
public class Main
{
	private Button button = new Button();	
	public Main() throws SecurityException, IllegalArgumentException, NoSuchMethodException, IllegalAccessException, InvocationTargetException
	{
		init();
	} 
	public void click()
	{
		System.out.println("Button clicked!");
	} 
	public void init() throws SecurityException,
			NoSuchMethodException, IllegalArgumentException,
			IllegalAccessException, InvocationTargetException
	{
		OnClickListenerHandler h = new OnClickListenerHandler(this);
		Method method = Main.class.getMethod("click", null);
		h.addMethod("onClick", method);
		Object clickProxy = Proxy.newProxyInstance(
				OnClickListener.class.getClassLoader(),
				new Class<?>[] { OnClickListener.class }, h);
		Method clickMethod = button.getClass().getMethod("setOnClickLisntener",
				OnClickListener.class);
		clickMethod.invoke(button, clickProxy);		
	}
 
	public static void main(String[] args) throws SecurityException,
			IllegalArgumentException, NoSuchMethodException,
			IllegalAccessException, InvocationTargetException
	{ 
		Main main = new Main();		
		main.button.click();
	} 
}
```
我们模拟按钮点击：调用main.button.click()，实际执行的却是Main的click方法
看init中，我们首先初始化了一个OnClickListenerHandler，把Main的当前实例传入，然后拿到Main的click方法，添加到OnClickListenerHandler中的Map中。
然后通过Proxy.newProxyInstance拿到OnClickListener这个接口的一个代理，这样执行这个接口的所有的方法，都会去调用OnClickListenerHandler的invoke方法。
但是呢？OnClickListener毕竟是个接口，也没有方法体~~那咋办呢？这时候就到我们OnClickListenerHandler中的Map中大展伸手了
```
@Override
public Object invoke(Object proxy, Method method, Object[] args)
throws Throwable
{ 
String methodName = method.getName();
Method realMethod = methods.get(methodName);
return realMethod.invoke(targetObject, args);
}
```
我们显示的把要执行的方法，通过键值对存到Map里面了，等调用到invoke的时候，其实是通过传入的方法名，得到Map中存储的方法，然后调用我们预设的方法~。

###Proxy原理分析
```
public static Object newProxyInstance(ClassLoader loader, 
            Class<?>[] interfaces, 
            InvocationHandler h) 
            throws IllegalArgumentException { 
 
    // 检查 h 不为空，否则抛异常
    if (h == null) { 
        throw new NullPointerException(); 
    } 
 
    // 获得与制定类装载器和一组接口相关的代理类类型对象
    Class cl = getProxyClass(loader, interfaces); 
 
    // 通过反射获取构造函数对象并生成代理类实例
    try { 
        Constructor cons = cl.getConstructor(constructorParams); 
        return (Object) cons.newInstance(new Object[] { h }); 
    } catch (NoSuchMethodException e) { throw new InternalError(e.toString()); 
    } catch (IllegalAccessException e) { throw new InternalError(e.toString()); 
    } catch (InstantiationException e) { throw new InternalError(e.toString()); 
    } catch (InvocationTargetException e) { throw new InternalError(e.toString()); 
    } 
}
```
动态代理真正的关键是在 getProxyClass 方法，该方法负责为一组接口动态地生成代理类类型对象,动态的生一个实现了一组接口的类。该方法总共可以分为四个步骤：
1. 对这组接口进行一定程度的安全检查，包括检查接口类对象是否对类装载器可见并且与类装载器所能识别的接口类对象是完全相同的，还会检查确保是 interface 类型而不是 class 类型。这个步骤通过一个循环来完成，检查通过后将会得到一个包含所有接口名称的字符串数组，记为 String[] interfaceNames。总体上这部分实现比较直观，所以略去大部分代码，仅保留留如何判断某类或接口是否对特定类装载器可见的相关代码。
通过 Class.forName 方法判接口的可见性
```
try { 
    // 指定接口名字、类装载器对象，同时制定 initializeBoolean 为 false 表示无须初始化类
    // 如果方法返回正常这表示可见，否则会抛出 ClassNotFoundException 异常表示不可见
    interfaceClass = Class.forName(interfaceName, false, loader); 
} catch (ClassNotFoundException e) { 
}
```
2. 从 loaderToCache 映射表中获取以类装载器对象为关键字所对应的缓存表，如果不存在就创建一个新的缓存表并更新到 loaderToCache。缓存表是一个 HashMap 实例，正常情况下它将存放键值对（接口名字列表，动态生成的代理类的类对象引用）。当代理类正在被创建时它会临时保存（接口名字列表，pendingGenerationMarker）。标记 pendingGenerationMarke 的作用是通知后续的同类请求（接口数组相同且组内接口排列顺序也相同）代理类正在被创建，请保持等待直至创建完成。
3. 动态创建代理类的类对象。首先是确定代理类所在的包，其原则如前所述，如果都为 public 接口，则包名为空字符串表示顶层包；如果所有非 public 接口都在同一个包，则包名与这些接口的包名相同；如果有多个非 public 接口且不同包，则抛异常终止代理类的生成。确定了包后，就开始生成代理类的类名，同样如前所述按格式“$ProxyN”生成。类名也确定了，接下来就是见证奇迹的发生 —— 动态生成代理类：
```
// 动态地生成代理类的字节码数组
byte[] proxyClassFile = ProxyGenerator.generateProxyClass( proxyName, interfaces); 
try { 
    // 动态地定义新生成的代理类
    proxyClass = defineClass0(loader, proxyName, proxyClassFile, 0, 
        proxyClassFile.length); 
} catch (ClassFormatError e) { 
    throw new IllegalArgumentException(e.toString()); 
}  
// 把生成的代理类的类对象记录进 proxyClasses 表
proxyClasses.put(proxyClass, null);
```
所有的代码生成的工作都由神秘的 ProxyGenerator 所完成了,而此类并没有公开。
4. 代码生成过程进入结尾部分，根据结果更新缓存表，如果成功则将代理类的类对象引用更新进缓存表，否则清楚缓存表中对应关键值，最后唤醒所有可能的正在等待的线程。

###代理类实现的原理：代码解释

```
// 假设需代理接口 Simulator 
public interface Simulator { 
    short simulate(int arg1, long arg2, String arg3) throws ExceptionA, ExceptionB;
}  
// 假设代理类为 SimulatorProxy, 其类声明将如下
final public class SimulatorProxy implements Simulator {  
    // 调用处理器对象的引用
    protected InvocationHandler handler;  
    // 以调用处理器为参数的构造函数
    public SimulatorProxy(InvocationHandler handler){ 
        this.handler = handler; 
    }  
    // 实现接口方法 simulate 
    public short simulate(int arg1, long arg2, String arg3) 
        throws ExceptionA, ExceptionB { 
        // 第一步是获取 simulate 方法的 Method 对象
        java.lang.reflect.Method method = null; 
        try{ 
            method = Simulator.class.getMethod( 
                "simulate", 
                new Class[] {int.class, long.class, String.class} );
        } catch(Exception e) { 
            // 异常处理 1（略）
        } 
 
        // 第二步是调用 handler 的 invoke 方法分派转发方法调用
        Object r = null; 
        try { 
            r = handler.invoke(this, 
                method, 
                // 对于原始类型参数需要进行装箱操作
                new Object[] {new Integer(arg1), new Long(arg2), arg3});
        }catch(Throwable e) { 
            // 异常处理 2（略）
        } 
        // 第三步是返回结果（返回类型是原始类型则需要进行拆箱操作）
        return ((Short)r).shortValue();
    } 
}
```
###扩展：Android使用动态代理和注解可以很轻松的实现对按钮的点击和设置