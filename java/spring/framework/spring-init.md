# Spring 初始化过程原理分析

## 基本信息
- 版本信息：5.0.8.RELEASE

## 引言
一般情况下，我们可以通过如下代码初始化一个Spring容器
```
    ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("application-context.xml");
```
改代码会取类路径下找到名为application-context.xml的配置文件，根据配置文件初始化得到ApplicationContext实例

## 初始化过程分析
### 初始化准备阶段
通过查看ClassPathXmlApplicationContext的源码知道，大部分构造方法都是调用了如下的构造方法：
```
	public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
```
该构造方法传入三个入参，分别为：
- configLocations，表示Spring配置文件路径的数组
- refresh，是否刷新容器，默认为true
- parent,用于初始化容器的父容器

构造的过程分三个步骤，如下：
1. 传入父容器并调用父类的构造方法，该方法在AbstractApplicationContext中实现，作用为设置父容器已经将父容器的属性值merge到当前容器
2. 设置配置文件的路径，在设置文件的过程中，对配置文件中用到的placeHolder进行了处理
3. refresh为容器初始化主逻辑，在AbstractApplicationContext实现

### 初始化主逻辑
容器的初始化主逻辑在AbstractApplicationContext的refresh方法实现，具体代码如下：
```
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			prepareRefresh();
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			prepareBeanFactory(beanFactory);
			try {
				postProcessBeanFactory(beanFactory);
				invokeBeanFactoryPostProcessors(beanFactory);
				registerBeanPostProcessors(beanFactory);
				initMessageSource();
				initApplicationEventMulticaster();
				onRefresh();
				registerListeners();
				finishBeanFactoryInitialization(beanFactory);
				finishRefresh();
			}catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}
				destroyBeans();
				cancelRefresh(ex);
				throw ex;
			}finally {
				resetCommonCaches();
			}
		}
	}
```
以上源码逐句分析如下：
#### prepareRefresh() refresh准备，源码如下：
```
	protected void prepareRefresh() {
		this.startupDate = System.currentTimeMillis();
		this.closed.set(false);
		this.active.set(true);

		if (logger.isInfoEnabled()) {
			logger.info("Refreshing " + this);
		}

		// Initialize any placeholder property sources in the context environment
		initPropertySources();

		// Validate that all properties marked as required are resolvable
		// see ConfigurablePropertyResolver#setRequiredProperties
		getEnvironment().validateRequiredProperties();

		// Allow for the collection of early ApplicationEvents,
		// to be published once the multicaster is available...
		this.earlyApplicationEvents = new LinkedHashSet<>();
	}
```

根据源码可知，该方法有一下逻辑
- 设置启动时间
- 设置关闭状态为false
- 设置激活装填为true
- 初始化initPropertySources
- 校验必须的属性值
- 新建earlyApplicationEvents对象用于存放初始化过程中的广播事件

#### ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory() 创建beanFactory 获取beanFactory对象，源码为：

```
	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (logger.isDebugEnabled()) {
			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
		}
		return beanFactory;
	}
```

#### prepareBeanFactory(beanFactory)； 源码如下.

```
	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// Tell the internal bean factory to use the context's class loader etc.
		beanFactory.setBeanClassLoader(getClassLoader());
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// Configure the bean factory with context callbacks.
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}
```

#### postProcessBeanFactory(beanFactory); 用于提供给子类实现

#### invokeBeanFactoryPostProcessors(beanFactory); 调用处理BeanFactory前的逻辑

#### registerBeanPostProcessors(beanFactory); 注册实例Bean前处理器

#### initMessageSource(); 初始化MessageSource

#### initApplicationEventMulticaster(); 初始化时间广播

#### onRefresh(); 用于实现类实现

#### registerListeners(); 注册Listeners，listener已经在prepareBeanFactory时读入

#### finishBeanFactoryInitialization(beanFactory); 主要为初始化所有Bean

#### finishRefresh(); 完成refresh的操作，代码为
```
	protected void finishRefresh() {
		// Clear context-level resource caches (such as ASM metadata from scanning).
		clearResourceCaches();

		// Initialize lifecycle processor for this context.
		initLifecycleProcessor();

		// Propagate refresh to lifecycle processor first.
		getLifecycleProcessor().onRefresh();

		// Publish the final event.
		publishEvent(new ContextRefreshedEvent(this));

		// Participate in LiveBeansView MBean, if active.
		LiveBeansView.registerApplicationContext(this);
	}
```

#### resetCommonCaches(); 重置缓存

## 总结
#### BeanFactoryPostProcessor和BeanPostProcessor
- BeanFactoryPostProcessor 初始化BeanFactory前的处理，此时，BeanFacctory对象已经创建，invokeBeanFactoryPostProcessors(beanFactory)触发。由于还未到初始化Bean的阶段，Spring无法使用自动注入。
- BeanPostProcessor 初始化Bean前处理，在finishBeanFactoryInitialization处理，每个Bean初始化时都会调用，可以对所有并进行统一操作。该类中可以使用自动注入。Spring可以自动调整Bean的初始化顺序。

#### 属性配置初始化时间很早，无法通过对应的属性Bean直接注入，但是可以通过Envirement获取。



