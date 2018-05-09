---
layout: post
title: Spring源码分析（4）Bean组件
date:   2017-10-04 19:15:10
categories:  Java
tags:  Spring
keywords: Spring
description: 
---
----------------------------------

## Bean组件
bean组件在Spring的Spring.springframework.beans包下，这个包主要解决了3个问题
* Bean的定义
* Bean的创建
* Bean的解析


## Bean的创建
我们主要关心Bean的创建，Spring Bean的创建是典型的工厂模式，它的顶级接口是BeanFactory。

BeanFactory中定义了getBean（）方法，可以从工厂中取到Bean的实例，当然是从接口的实现类中获取。

```
public interface BeanFactory {
	String FACTORY_BEAN_PREFIX = "&";
	
	//getBean方法，返回指定bean的实例，该实例可以是共享的或独立的。
	Object getBean(String name) throws BeansException;
	...
	
	//是否存在具有给定名称的bean
	boolean containsBean(String name);
	
        //这两个接口是判断Bean的作用于，单例还是多例
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
	
        //检查具有给定名称的bean是否与指定的类型匹配。
	boolean isTypeMatch(String name, @Nullable Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	//确定具有给定名称的bean的类型
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	
	//返回给定bean名称的别名，如果有的话.
	String[] getAliases(String name);

}
```
在BeanFactory里只对IOC容器的基本行为作了定义，根本不关心你的bean是如何定义怎样加载的。正如我们只关心工厂里得到什么的产品对象，至于工厂是怎么生产这些对象的，这个基本的接口不关心。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/beanfactory.png)

可以看到，BeanFactory的三个子类ListableBeanFactory,HierarchicalBeanFactory和AutowireCapableBeanFactory,分三类主要是为了区分Spring内部对象的传递和转化过程，对对象的数据访问做限制。
ListableBeanFactory接口表示Bean是可列表的
HierarchicalBeanFactory表示Bean是有继承关系的，每个Bean可能有父Bean
AutowireCapableBeanFactory接口定义Bean的自动装配规则

而要知道工厂是如何产生对象的，我们需要看具体的IOC容器实现，spring提供了许多IOC容器的实现。比如XmlBeanFactory，ClasspathXmlApplicationContext等。其中XmlBeanFactory就是针对最基本的ioc容器的实现，这个IOC容器可以读取XML文件定义的BeanDefinition（XML文件中对bean的描述）。

而ApplicationContext是Spring提供的一个高级的IoC容器，它除了能够提供IoC容器的基本功能外，还为用户提供了以下的附加服务。

从ApplicationContext接口的实现，我们看出其特点：（ApplicationContext源码见Context组件文章）
*   支持信息源，可以实现国际化。（实现MessageSource接口）
*   访问资源。(实现ResourcePatternResolver接口，这个后面要讲)
*   支持应用事件。(实现ApplicationEventPublisher接口)

## Bean的定义
Bean的定义主要由BeanDefinition描述，Spring成功解析< bean />节点后，在Spring的内部它就被转化成了BeanDefinition对象，后续的操作都围绕BeanDefinition来进行。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/BeanDefinition.png)

```
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

	String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
	String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;
	int ROLE_APPLICATION = 0;
	int ROLE_SUPPORT = 1;
	int ROLE_INFRASTRUCTURE = 2;


	// Modifiable attributes

	//为这个BeanDefinition的父类BeanDefinition设置名字
	void setParentName(@Nullable String parentName);

	//返回这个BeanDefinition的父类BeanDefinition的名字
	@Nullable
	String getParentName();

	//指定此beanDefinition的bean class名称
	void setBeanClassName(@Nullable String beanClassName);

	//返回这个BeanDefinition的当前BeanClass名称
	@Nullable
	String getBeanClassName();

	//覆盖此bean的目标作用域，并指定一个新的作用域名称。
	void setScope(@Nullable String scope);

	//返回当前bean目标作用域的名称
	@Nullable
	String getScope();
	
        #懒加载的get&set方法
	//设置当前Bean是否应该懒加载
	void setLazyInit(boolean lazyInit);
	//返回当前bean是否是懒加载
	boolean isLazyInit();
        
	#依赖关系设置
	//设置这个bean依赖被初始化的bean的名称。beanFactory将保证这些bean首先被初始化。
	void setDependsOn(@Nullable String... dependsOn); 
	//返回这个bean依赖的bean名称。
	@Nullable
	String[] getDependsOn();

	/**
	 * Set whether this bean is a candidate for getting autowired into some other bean.
	 * <p>Note that this flag is designed to only affect type-based autowiring.
	 * It does not affect explicit references by name, which will get resolved even
	 * if the specified bean is not marked as an autowire candidate. As a consequence,
	 * autowiring by name will nevertheless inject a bean if the name matches.
	 */
	void setAutowireCandidate(boolean autowireCandidate);

	/**
	 * Return whether this bean is a candidate for getting autowired into some other bean.
	 */
	boolean isAutowireCandidate();

	/**
	 * Set whether this bean is a primary autowire candidate.
	 * <p>If this value is {@code true} for exactly one bean among multiple
	 * matching candidates, it will serve as a tie-breaker.
	 */
	void setPrimary(boolean primary);

	/**
	 * Return whether this bean is a primary autowire candidate.
	 */
	boolean isPrimary();

        #定义创建该Bean对象的工厂类
	//指定要使用的FactoryBean的名称，
	void setFactoryBeanName(@Nullable String factoryBeanName);
	//返回FactoryBean的名称
	@Nullable
	String getFactoryBeanName();

	#创建该Bean对象的工厂方法
	//
	void setFactoryMethodName(@Nullable String factoryMethodName);
	//返回Factoryf方法名称
	@Nullable
	String getFactoryMethodName();

	/**
	 * Return the constructor argument values for this bean.
	 * <p>The returned instance can be modified during bean factory post-processing.
	 * @return the ConstructorArgumentValues object (never {@code null})
	 */
	ConstructorArgumentValues getConstructorArgumentValues();

	/**
	 * Return if there are constructor argument values defined for this bean.
	 * @since 5.0.2
	 */
	default boolean hasConstructorArgumentValues() {
		return !getConstructorArgumentValues().isEmpty();
	}

	//返回要应用于bean的新实例的属性值。
	MutablePropertyValues getPropertyValues();

	//如果存在为此Bean定义的属性值，则返回。
	default boolean hasPropertyValues() {
		return !getPropertyValues().isEmpty();
	}


	#当前Bean的基本特性
	//是否是单例
	boolean isSingleton();
	//是否是多例
	boolean isPrototype();
	//是否是抽象类
	boolean isAbstract();

	/**
	 * Get the role hint for this {@code BeanDefinition}. The role hint
	 * provides the frameworks as well as tools with an indication of
	 * the role and importance of a particular {@code BeanDefinition}.
	 * @see #ROLE_APPLICATION
	 * @see #ROLE_SUPPORT
	 * @see #ROLE_INFRASTRUCTURE
	 */
	int getRole();

	/**
	 * Return a human-readable description of this bean definition.
	 */
	@Nullable
	String getDescription();

	/**
	 * Return a description of the resource that this bean definition
	 * came from (for the purpose of showing context in case of errors).
	 */
	@Nullable
	String getResourceDescription();

	/**
	 * Return the originating BeanDefinition, or {@code null} if none.
	 * Allows for retrieving the decorated bean definition, if any.
	 * <p>Note that this method returns the immediate originator. Iterate through the
	 * originator chain to find the original BeanDefinition as defined by the user.
	 */
	@Nullable
	BeanDefinition getOriginatingBeanDefinition();

}

```
 仔细观看，能发现beanDefinition中有两个方法分别是String[] getDependsOn()和void setDependsOn(String[] dependsOn)，这两个方法就是获取依赖的beanName和设置依赖的beanName，这样就好办了，只要我们有一个BeanDefinition，就可以完全的产生一个完整的bean实例。
