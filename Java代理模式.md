# 代理分为：静态代理、动态代理、Cglib代理

```
代理应用在spring的事务处理中，处理异常回滚等。
在Spring的AOP编程中:
 如果加入容器的目标对象有实现接口,用JDK代理
 如果目标对象没有实现接口,用Cglib代理
```

```
/**
 * 接口
 */
public interface IUserDao {

    void save();
}

```
```
/**
 * 接口实现
 */
public class UserDao implements IUserDao {
    public void save() {
      ///TODO
    }
}
```

## 1、静态代理
```
 静态代理在使用时,需要定义接口或者父类,被代理对象与代理对象一起实现相同的接口或者是继承相同父类.
```
```
/**
 * 静态代理
 */
public class UserDaoProxy implements IUserDao{
    //接收保存目标对象
    private IUserDao target;
    public UserDaoProxy(IUserDao target){
        this.target=target;
    }

    public void save() {
        ///TODO
        target.save();//执行目标对象的方法
        ///TODO
    }
}
```
```
/**
 * 测试
 */
public class App {
    public static void main(String[] args) {
        //目标对象
        UserDao target = new UserDao();

        //代理对象,把目标对象传给代理对象,建立代理关系
        UserDaoProxy proxy = new UserDaoProxy(target);

        proxy.save();//执行的是代理的方法
    }
}
```
## 2、动态代理
```
代理类所在包:java.lang.reflect.Proxy
JDK实现代理只需要使用newProxyInstance方法
```
```
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,InvocationHandler h )
```

```
/**
 * 创建动态代理对象
 */
public class ProxyFactory{

    private Object target;
    public ProxyFactory(Object target){
        this.target=target;
    }

    public Object getProxyInstance(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        ///TODO
                        Object rt = method.invoke(target, args);
                        ///TODO
                        return rt;
                    }
                }
        );
    }

}
```
```
public class App {
    public static void main(String[] args) {
    
        IUserDao target = new UserDao();
        IUserDao proxy = (IUserDao) new ProxyFactory(target).getProxyInstance();

        proxy.save();
    }
}
```
## 3、Cglib代理

``` 
Spring的核心包中已经包括了Cglib功能,引入spring-core-3.2.5.jar

代理的类不能为final,否则报错
目标对象的方法如果为final/static,那么就不会被拦截,即不会执行目标对象额外的业务方法.


public class UserDao {

    public void save() {
      ///TODO
    }
}

public class ProxyFactory implements MethodInterceptor {

    private Object target;
    public ProxyFactory(Object target) {
        this.target = target;
    }

    //给目标对象创建一个代理对象
    public Object getProxyInstance(){
        //1.工具类
        Enhancer en = new Enhancer();
        //2.设置父类
        en.setSuperclass(target.getClass());
        //3.设置回调函数
        en.setCallback(this);
        //4.创建子类(代理对象)
        return en.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        ///TODO
        Object returnValue = method.invoke(target, args);
        ///TODO
        return returnValue;
    }
}

/**
 * 测试类
 */
public class App {

    @Test
    public void test(){
    
        UserDao target = new UserDao();
        UserDao proxy = (UserDao)new ProxyFactory(target).getProxyInstance();

        proxy.save();
    }
}
```
