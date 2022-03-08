# Spring

## 谈谈对IOC的理解

IOC也就是控制反转，原来的对象是使用者进行控制，有了Spring容器之后由Spring容器进行管理。

DI ：依赖注入，把对应属性的值注入进去例如@Autowired，构造器注入等

容器：使用map存储对象，在spring中一般存在三级缓存，一般情况下我们取对象从singleonObject中去取

整个bean的生命周期，从创建到使用到销毁的过程是由容器来管理

## IOC的底层实现

1：通过createBeanFactory创建bean工厂

2：开始循环创建对象，因为同期中的bean都是单例的。所以优先通过getBean，doGetBean从容器中查找

3：找不到的话，通过createBean和doCreateBean方法，通过反射创建对象。一般都是无参构造方法

4：对对象的属性进行填充

5:  进行其他初始化操作

## bean的生命周期

流程图如下

![](https://raw.githubusercontent.com/409582940/notes/main/images/20220307172941.png)

SpringBean是形成Spring应用主干的Java对象，可以被SpringIOC容器初始化，装配和管理。这些bean通过配置中的元数据创建

在传统的Java应用中，创建对象是用new来创建，当不需要时由jvm自动进行回收

而SpringBean的生命周期可以分为以下几个阶段：

1：Spring对bean进行实例化：反射的方式生成对象

2：填充bean的属性。（循环依赖的问题）

3：调用aware接口相关的方法

如果bean实现了BeanNameAware接口，Spring将beanID传递给setBeanName方法（让bean获得自身的id属性）

如果bean实现了BeanFactoryAware接口，Spring调用setBeanFactory的方法，将BeanFactory容器实例传入

如果bean实现了ApplicationContextAware接口，Spring调用setApplicationContextAware方法，将bean所在的应用上下文的引用传入进来

4：调用BeanPostProcessor的前置处理方法，Spring调用post-ProcessBeforeInitialization方法

5：调用initmethod方法invokeInitMethod()：如果bean实现了InitializingBean接口，Spring将调用after-PropertiesSet方法。如果bean使用initmethod声明了初始化的方法，该方法也会被调用

6：调用BeanPostProcessor的后置处理方法，Spring将调用它们的post-ProcessAfterInitialization()方法（Bean的初始化方法（init-method）被容器调用之后执行），spring的AOP就是在在此处实现

7：此时，bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁；（可以通过getBean获得）

8：如果bean实现了DisposableBean接口，Spring将调用它的destroy()接口方法。同样，如果bean使用destroy-method声明了销毁方法，该方法也会被调用。

## Spring是如何解决循环依赖的问题的
关键词：**三级缓存**、**提前暴露对象** **AOP**

首先，循环依赖就是在容器创建对象的过程中，A中有依赖B的属性，B中由依赖A中的属性
但是仔细思考的话，我们可以发现在创建B时候已经有A对象，只不过这个对象不完整，只完成了实例化没有完成初始化。所以关键就是可不可以将对象的实例化和初始化分开

当所有的对象都完成实例化和初始化操作之后，还要把完整的对象放到容器中。此时容器中的对象存在两个状态，分别是已经完成实例化，没有初始化和完成实例化和初始化的
不同的状态需要不同的容器进行存储，此时就有了一级缓存和二级缓存。
如果一级缓存中已经存在，二级缓存就不会存在。因为状态是互斥的
在查询的过程中，查询顺序是一级缓存，二级缓存，三级缓存中查找的
所以，一级缓存中存储的是完整的对象，二级缓存，三级缓存中的对象是不完整的
### 为什么需要三级缓存
三级缓存的value是`ObjectFactory`，存在的意义是保证在整个同期的运行过程中同名的对象只能有这一个
普通对象和代理对象是不能同时出现在容器中的，当一个对象需要被代理时，就要用代理对象覆盖掉原本的普通对象。但是在实际的调用过程中，不知道什么时候对象被使用。所以就要求某个对象被调用的时候，优先判断此对象是否需要代理，类似一种回调机制的体现。这就是三级缓存存在的意义。因为在Spring AOP功能，就是通过动态代理类来实现的，最后我们使用的也就是代理类的实例而不是原始类的实例

bean创建流程图如下

![](https://raw.githubusercontent.com/409582940/notes/main/images/20220307170744.png)

解决问题的核心就是实例化和初始化分开操作，这也就是解决循环依赖问题的关键

一句话总结：主要是Spring的三级缓存解决了这个问题，并且构造器注入无法通过此方式解决。Spring有一个注解@Lazy是为了解决构造器注入循环依赖的问题

具体怎么解决如下

伪代码：

``` java

// 一级缓存

private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

// 二级缓存

private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

// 三级缓存

private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

  

protected Object getBean(final String beanName) {

// !以下为getSingleton逻辑！

// 先从一级缓存获取

Object single = singletonObjects.get(beanName);

if (single != null) {

return single;

}

// 再从二级缓存获取

single = earlySingletonObjects.get(beanName);

if (single != null) {

return single;

}

// 从三级缓存获取objectFactory

ObjectFactory<?> objectFactory = singletonFactories.get(beanName);

if (objectFactory != null) {

single = objectFactory.get();

// 升到二级缓存

earlySingletonObjects.put(beanName, single);

singletonFactories.remove(beanName);

return single;

}

// !以上为getSingleton逻辑！

// ！以下为doCreateBean逻辑

// 缓存完全拿不到，需要创建

// 创建实例

Object beanInstance = createBeanInstance(beanName);

// 实例创建之后，放入三级缓存

singletonFactories.put(beanName, () -> return beanInstance);

// 依赖注入，会触发依赖的bean的getBean方法

populateBean(beanName, beanInstance);

// 初始化方法调用

initializeBean(beanName, beanInstance);

// 依赖注入完之后，如果二级缓存有值，说明出现了循环依赖

// 这个时候直接取二级缓存中的bean实例

Object earlySingletonReference = earlySingletonObjects.get(beanName);

if (earlySingletonReference != null) {

beanInstance = earlySingletonObject;

}

// ！以上为doCreateBean逻辑

// 从二三缓存移除，放入一级缓存

singletonObjects.put(beanName, beanInstance);

earlySingletonObjects.remove(beanName);

singletonFactories.remove(beanName);

return beanInstance;

}

```

