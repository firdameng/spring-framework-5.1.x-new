

# spring bean创建流程

org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization  

org.springframework.beans.factory.config.ConfigurableListableBeanFactory#preInstantiateSingletons 初始化所有剩下的非懒加载的单例bean

org.springframework.beans.factory.support.AbstractBeanFactory#getBean

org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean



org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean



org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBeanInstance 使用无参构造创建bean,但未设置属性

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean 属性填充（即填充依赖的bean）

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean 调用setBeanName  方法，setBeanFactory⽅法  ，调用BeanPostProcessorsBeforeInitialization方法（如果bean实现了applicationaware接口，该接口有个特殊的ApplicationContextAwareProcessor ,那么在此阶段作为第一个BeanPostProcessor被调用，即调用setsetApplicationContext 方法，在prepareBeanFactory方法中首次添加，保证了第一性），

调用afterPropertiesSet方法，调用定制的初始化方法，调用BeanPostProcessorsafterInitialization方法

# bean factory创建流程

org.springframework.context.support.AbstractApplicationContext#refresh

org.springframework.context.support.AbstractApplicationContext#obtainFreshBeanFactory  获取BeanFactory；默认实现是DefaultListableBeanFactory，加载BeanDefition 并注册到 BeanDefitionRegistry

org.springframework.context.support.AbstractRefreshableApplicationContext#refreshBeanFactory

这里创建beanfactory, 接着加载bean定义



org.springframework.beans.factory.support.AbstractBeanDefinitionReader#loadBeanDefinitions

org.springframework.beans.factory.xml.XmlBeanDefinitionReader#doLoadBeanDefinitions



# AbstractApplicationContext#refresh方法 流程

1.  prepareRefresh  设置Spring容器的启动时间，开启活跃状态，撤销关闭状态
2.  **<u>obtainFreshBeanFactory 获取BeanFactory 加载BeanDefition 并注册到 BeanDefitionRegistry</u>**
3. prepareBeanFactory BeanFactory的预准备工作,设置context的类加载器，**设置ApplicationContextAware的后置处理器**
4. postProcessBeanFactory  上下文子类 对 beanfactory的准备工作完成后的后置处理
5. **invokeBeanFactoryPostProcessors 实例化实现了BeanFactoryPostProcessor接口的Bean（从bean定义中拿到类型为BeanFactoryPostProcessor的bean名称，然后通过getbean实例bean) , 并调用接口方法**
6. registerBeanPostProcessors  注册BeanPostProcessor（Bean的后置处理器），在创建bean的前后等执行
7. initMessageSource 初始化MessageSource组件（做国际化功能；消息绑定，消息解析）
8. initApplicationEventMulticaster  初始化事件派发器
9. onRefresh 子类重写这个方法，在容器刷新的时候可以自定义逻辑；如创建Tomcat，Jetty等WEB服务器
10. registerListeners 注册应用的监听器。就是注册实现了ApplicationListener接口的监听器bean
11.  **finishBeanFactoryInitialization 初始化所有剩下的非懒加载的单例bean**
12. finishRefresh 完成context的刷新。主要是调用LifecycleProcessor的onRefresh()方法，并且发布事件（ContextRefreshedEvent）

```java
public void refresh() throws BeansException, IllegalStateException {
		// 对象锁加锁
		synchronized (this.startupShutdownMonitor) {
			/*
				Prepare this context for refreshing.
			 	刷新前的预处理
			 	表示在真正做refresh操作之前需要准备做的事情：
					设置Spring容器的启动时间，
					开启活跃状态，撤销关闭状态
					验证环境信息里一些必须存在的属性等
			 */
			prepareRefresh();

			/*
				Tell the subclass to refresh the internal bean factory.
			 	获取BeanFactory；默认实现是DefaultListableBeanFactory
                加载BeanDefition 并注册到 BeanDefitionRegistry
			 */
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			/*
				Prepare the bean factory for use in this context.
				BeanFactory的预准备工作（BeanFactory进行一些设置，比如context的类加载器等）
			 */
			prepareBeanFactory(beanFactory);

			try {
				/*
					Allows post-processing of the bean factory in context subclasses.
					BeanFactory准备工作完成后进行的后置处理工作
				 */
				postProcessBeanFactory(beanFactory);

				/*
					Invoke factory processors registered as beans in the context.
					实例化实现了BeanFactoryPostProcessor接口的Bean，并调用接口方法
				 */
				invokeBeanFactoryPostProcessors(beanFactory);

				/*
					Register bean processors that intercept bean creation.
					注册BeanPostProcessor（Bean的后置处理器），在创建bean的前后等执行
				 */
				registerBeanPostProcessors(beanFactory);

				/*
					Initialize message source for this context.
					初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；
				 */
				initMessageSource();

				/*
					Initialize event multicaster for this context.
					初始化事件派发器
				 */
				initApplicationEventMulticaster();

				/*
					Initialize other special beans in specific context subclasses.
					子类重写这个方法，在容器刷新的时候可以自定义逻辑；如创建Tomcat，Jetty等WEB服务器
				 */
				onRefresh();

				/*
					Check for listener beans and register them.
					注册应用的监听器。就是注册实现了ApplicationListener接口的监听器bean
				 */
				registerListeners();

				/*
					Instantiate all remaining (non-lazy-init) singletons.
					初始化所有剩下的非懒加载的单例bean
					初始化创建非懒加载方式的单例Bean实例（未设置属性）
                    填充属性
                    初始化方法调用（比如调用afterPropertiesSet方法、init-method方法）
                    调用BeanPostProcessor（后置处理器）对实例bean进行后置处理
				 */
				finishBeanFactoryInitialization(beanFactory);

				/*
					Last step: publish corresponding event.
					完成context的刷新。主要是调用LifecycleProcessor的onRefresh()方法，并且发布事件（ContextRefreshedEvent）
				 */
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```

# spring如何解决 存在aop对象时 的 循环依赖

![image-20211003101230978](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003101230978.png)

![Snipaste_2021-10-02_22-31-32](.\img\Snipaste_2021-10-02_22-31-32.png)



1. 放入三级缓存

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingletonFactory

依赖的itbean

![image-20211002224103527](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002224103527.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingletonFactory

放入单例工厂的是lagouBean的工厂对象

![image-20211002224336109](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002224336109.png)

2. lagoubean属性填充

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean

![image-20211002225119029](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002225119029.png)



![image-20211002225206859](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002225206859.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyPropertyValues

![image-20211002225336148](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002225336148.png)

org.springframework.beans.factory.support.BeanDefinitionValueResolver#resolveValueIfNecessary

![image-20211002225433500](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002225433500.png)

org.springframework.beans.factory.support.BeanDefinitionValueResolver#resolveReference

![image-20211002225458877](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002225458877.png)

org.springframework.beans.factory.support.AbstractBeanFactory#getBean(java.lang.String)

org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean

![image-20211002230100859](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230100859.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton

![image-20211002230046067](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230046067.png)

![image-20211002230121653](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230121653.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean

![image-20211002230157011](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230157011.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean

![image-20211002230236987](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230236987.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingletonFactory

![image-20211002230311716](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230311716.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean

![image-20211002230335907](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230335907.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean

![image-20211002230410930](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230410930.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyPropertyValues

![image-20211002230437386](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230437386.png)

org.springframework.beans.factory.support.BeanDefinitionValueResolver#resolveValueIfNecessary

![image-20211002230458538](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230458538.png)

org.springframework.beans.factory.support.BeanDefinitionValueResolver#resolveReference

![image-20211002230524899](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230524899.png)

org.springframework.beans.factory.support.AbstractBeanFactory#getBean(java.lang.String)

org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean

![image-20211002230644170](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230644170.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String, boolean)

![image-20211002230837634](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230837634.png)

![image-20211002230954087](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002230954087.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#getEarlyBeanReference

这里拿到 lagoubean的代理对象，也就是从lagou工厂bean获取半成品的lagoubean

正常情况下，代理对象的产生 在属性设置后的bean初始化过程中的后置处理阶段。org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean方法中的 applyBeanPostProcessorsAfterInitialization方法。

![image-20211002231226284](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002231226284.png)

org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#getEarlyBeanReference

![image-20211002231329850](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002231329850.png)

![image-20211002231715937](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002231715937.png)

![image-20211002231742529](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002231742529.png)

org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#wrapIfNecessary

![image-20211002232945513](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002232945513.png)

org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#createProxy

![image-20211002233119265](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002233119265.png)

org.springframework.aop.framework.ProxyFactory#getProxy(java.lang.ClassLoader)

org.springframework.aop.framework.CglibAopProxy#getProxy(java.lang.ClassLoader)

![image-20211002233328417](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002233328417.png)

org.springframework.aop.framework.ObjenesisCglibAopProxy#createProxyClassAndInstance

![image-20211002233439768](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002233439768.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#getEarlyBeanReference

这里返回的已经是一个代理对象了

![image-20211002233638075](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002233638075.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String, boolean)

![image-20211002233830098](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002233830098.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyPropertyValues

![image-20211002234008961](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002234008961.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean

itbean属性设置完毕，开始调用构造函数初始化，已经后置处理器处理

![image-20211002234104728](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211002234104728.png)



![image-20211003001118469](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003001118469.png)

![image-20211003001424045](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003001424045.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])

![image-20211003001448006](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003001448006.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton

![image-20211003001552237](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003001552237.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingleton

直接将itbean从3级缓存挪到1级缓存了

![image-20211003001634908](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003001634908.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyPropertyValues

回到lagoubean处理属性这边

![image-20211003001756124](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003001756124.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean

![image-20211003001901428](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003001901428.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String, boolean)

![image-20211003002025686](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003002025686.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String, org.springframework.beans.factory.ObjectFactory<?>)

在这里添加将lagoubean从2级缓存 挪到1级缓存

![image-20211003002140781](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003002140781.png)

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingleton

![image-20211003002342141](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003002342141.png)



# aop 正常创建流程

![image-20211003101447461](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003101447461.png)

如果该对象不设置循环依赖，aop对象正常创建 一定是在 设置属性（填充依赖的对象）后，**对象初始化过程中的后置处理阶段。**

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)

![image-20211003101513433](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003101513433.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization

![image-20211003101534608](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003101534608.png)

org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#postProcessAfterInitialization

![image-20211003101733438](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003101733438.png)

org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#wrapIfNecessary

![image-20211003101803177](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003101803177.png)

org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#createProxy

![image-20211003101831509](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003101831509.png)



# factoryBean 以 mybatis中的sqlSessionFactoryBean为例



userMapper- > 

mapperFactoryBean->getObject-》 获取到userMapper的工厂bean, 调用getObject生成实例

sqlsessionTemplate.getMapper-> 

mapperRegistry.getMapper->

mapperProxyFactory.newInstance



org.mybatis.spring.mapper.MapperFactoryBean

MapperFactoryBean 有个属性为 sqlSessionFactory,因此创建时，会填充sqlSessionFactory属性，

获取sqlSessionFactory时，会根据名称先 拿到sqlSessionFactoryBean，调用getobject得到sqlSessionFactory，

![image-20211003141642603](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003141642603.png)

![image-20211003141411403](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003141411403.png)



org.mybatis.spring.SqlSessionFactoryBean#getObject

![image-20211003142031763](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003142031763.png)

如果一开始就没有SqlSessionFactory，调用

org.mybatis.spring.SqlSessionFactoryBean#afterPropertiesSet

org.mybatis.spring.SqlSessionFactoryBean#buildSqlSessionFactory

org.apache.ibatis.session.SqlSessionFactoryBuilder#build(org.apache.ibatis.session.Configuration)

最终返回SqlSessionFactory

![image-20211003142528286](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003142528286.png)

拿到sqlsessionFactory后，开始设置MapperFactoryBean的属性sqlsessionFactory，

![image-20211003143806021](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003143806021.png)

![image-20211003143815505](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003143815505.png)

![image-20211003143901914](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003143901914.png)

其实本质上MapperFactoryBean 继承sqlsessionFactory ，从而拥有  SqlSessionTemplate，所以直接从类继承上看不出MapperFactoryBean 为啥拥有sqlSessionFactory属性，



org.mybatis.spring.mapper.ClassPathMapperScanner#processBeanDefinitions

- 未处理前bd定义

![image-20211003155706295](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003155706295.png)

- 处理后，bd定义

每一个mapper接口都会被扫描成一个BeanDefinition，这个BD开始会被强制设置成MapperFactoryBean类型，并且sqlSessioinFactory放进了bd的属性里

![image-20211003155215519](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003155215519.png)



accountMapper首先生成MapperFactoryBean工厂对象放到3级缓存，再处理 填充属性sqlSessionFactory



org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean

![image-20211003162655119](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003162655119.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean

![image-20211003162643416](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003162643416.png)

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyPropertyValues

![image-20211003162627384](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003162627384.png)

org.springframework.beans.BeanWrapperImpl.BeanPropertyHandler#setValue

![image-20211003162452288](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003162452288.png)

org.mybatis.spring.support.SqlSessionDaoSupport#setSqlSessionFactory

![image-20211003162502656](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003162502656.png)



这样MapperFactoryBean的sqlSessionTemplate就有值了

![image-20211003162759305](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003162759305.png)



那么accoutMapper 作为 MapperFactoryBean的工作就完成了，接着service层依赖mapper时，生成mapper代理对象。

这里我们先**根据依赖accoutMapper 的名称作为key**, 拿到beanFactory中的 MapperFactoryBean, 判断发现是工厂bean,接着去getObject方法获得最终bean,这时候就会生成代理对象。

![image-20211003163636837](C:\Users\CaiWencheng\AppData\Roaming\Typora\typora-user-images\image-20211003163636837.png)







- 这是根据类型获取bean的流程

我们调用getBean(Class requiredType)方法根据类型来获取容器中的bean的时候，对应我们的例子就是：根据类型com.zkn.spring.learn.service.FactoryBeanService来从Spring容器中获取Bean(首先明确的一点是在Spring容器中没有FactoryBeanService类型的BeanDefinition。但是却有一个Bean和FactoryBeanService这个类型有一些关系)。Spring在根据type去获取Bean的时候，会先获取到beanName。**获取beanName的过程是：先循环Spring容器中的所有的beanName，然后根据beanName获取对应的BeanDefinition，如果当前bean是FactoryBean的类型，则会从Spring容器中根据beanName获取对应的Bean实例，接着调用获取到的Bean实例的getObjectType方法获取到Class类型，判断此Class类型和我们传入的Class是否是同一类型。如果是则返回测beanName**，对应到我们这里就是：根据factoryBeanLearn获取到FactoryBeanLearn实例，调用FactoryBeanLearn的getObjectType方法获取到返回值FactoryBeanService.class。和我们传入的类型一致，所以这里获取的beanName为factoryBeanLearn。换句话说这里我们把factoryBeanLearn这个beanName映射为了：FactoryBeanService类型。即FactoryBeanService类型对应的beanName为factoryBeanLearn
————————————————
版权声明：本文为CSDN博主「木叶之荣」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/zknxx/article/details/79572387