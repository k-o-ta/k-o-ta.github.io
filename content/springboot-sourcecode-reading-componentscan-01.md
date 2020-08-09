+++
title = "Spring bootのコンポーネントスキャンのソースコードリーディングのログ"
date = 2020-08-10
tags = []
+++

spring bootを使うにあたってコンポーネントスキャン周りを理解したかった
## 登場人物
* コンポーネント: アプリケーションのパーツ(リポジトリの実装とかアルゴリズムの実装とか)
  * Springの世界ではDIコンテナで管理されているコンポーネントはBeanと呼ばれる(Bean定義じゃなくてBean実装)
* DIコンテナ: コンポーネントを登録する場所
  * Configuration: コンポーネントをDIコンテナに登録するときに使用される設定ファイル
    * Springの世界ではBean定義と呼ばれる
      * コンポーネントを実装しているコードをBean定義と呼べばいいのにと思ったが、[beanはmetadataとともに生成される](https://www.tutorialspoint.com/spring/spring_bean_definition.htm)とのことで、コンポーネント(POJO)をDIコンテナから使えるようにbeanとして定義するconfigurationのことをBean定義と呼ぶと理解した。
  * ApplicationContext: DIコンテナを触るためのインターフェース
* アプリケーション: コンポーネントの利用者
  * DIコンテナからBeanをルックアップする

spring bootのエントリーポイント
```java
// SpringBootMyApplication.java
@SpringBootApplication
public class SpringBootMyApplication {
  public static void main(String[] args) {
    SpringApplication.run(SpringBootMyApplication.class, args);
  }
}
```


spring bootでは`@SpringBootApplication`アノテーションが重要な役割を持つ([ソース](https://github.com/spring-projects/spring-boot/blob/v2.3.2.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/SpringBootApplication.java))
```java
// SpringBootApplication.java
// 省略
@SpringBootConfiguration
@ComponentScan
// 省略
public @interface SpringBootApplication {
  // 省略
}
```
`@SpringBootApplication`は`@ComponentScan`を継承している。`@CompnenntScan`はSpringで定義されており、はvalue属性に指定したパッケージ(デフォルトでは付与されたクラスと同じパッケージ)配下のコンポーネントをスキャンしてBeanとして登録できる。たとえば、以下のように書く
```java
@Configuration // configファイルであることの宣言
@ComponentScan("com.myexmple.app") // コンポーネントスキャン
public class AppConfig {
  @Bean // メソッド名: bean名, 返り値: beanのインスタンス
  MyRepository myRepository () {
    return new MyRepositoryImpl();
  }
  @Bean
  MyService myService(MyRepository myRepository) { // 他のコンポーネントを注入できる
    return new MyServiceImpl(myRepository);
  }
  @Bean
  MyService myService2 () {
    return new MyServiceImpl2(); // 同じインターフェースを満たした違う実装
  }
}
```
ConfigurationはBean定義でもあるので`@Bean`というアノテーションを付けるとコンポーネントスキャンしたファイルをBeanとしてDIコンテナに登録してくれるが、Spring BootではコンポーネントのPOJOに`@Component`のようなアノテーションを付けることでConfiguration側で`@Bean`を用いたメソッドを作らなくてもDIコンテナに登録されるようになる。(この仕組はSpringのもの)

`@ComponentScan`の動きを見ていく

まずエントリーポイントで`SpringApplication.run(SpringBootMyApplication.class, args);`が実行される
[ソース](https://github.com/spring-projects/spring-boot/blob/v2.3.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L1225-L1227)
```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class<?>[] { primarySource }, args);
}
```
上記のrunは[以下](https://github.com/spring-projects/spring-boot/blob/v2.3.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L1236-L1238)を呼び出す
```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return new SpringApplication(primarySources).run(args);
}
```
その後[以下](https://github.com/spring-projects/spring-boot/blob/v2.3.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L298-L337)が呼ばれる
```java
public ConfigurableApplicationContext run(String... args) {
  // 省略
  ConfigurableApplicationContext context = null;
  // 省略
  try {
    // 省略
    context = createApplicationContext();
    prepareContext(context, environment, listeners, applicationArguments, printedBanner);
    refreshContext(context);
    // 省略
  }
  // 省略
}
```
`createApplicationContext()`では [`ServletWebServerApplicationContext`](https://github.com/spring-projects/spring-boot/blob/v2.3.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/web/servlet/context/ServletWebServerApplicationContext.java)か、[`AnnotationConfigReactiveWebServerApplicationContext`](https://github.com/spring-projects/spring-boot/blob/v2.3.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/web/reactive/context/AnnotationConfigReactiveWebServerApplicationContext.java)か、[`AnnotationConfigApplicationContext`](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/AnnotationConfigApplicationContext.java)のいずれかのクラスが生成される。前者2つはSpringBootで最後の一つはSpringで定義されている。いずれも`AbstractApplicationContext`を継承している

refreshContextの定義は[以下](https://github.com/spring-projects/spring-boot/blob/v2.3.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L396-L406)の通り
```java
private void refreshContext(ConfigurableApplicationContext context) {
  refresh((ApplicationContext) context);
  // 省略
}
```

そして[refresh](https://github.com/spring-projects/spring-boot/blob/v2.3.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L741-L759)が呼ばれる
```java
protected void refresh(ApplicationContext applicationContext) {
  Assert.isInstanceOf(ConfigurableApplicationContext.class, applicationContext);
  refresh((ConfigurableApplicationContext) applicationContext);
}

protected void refresh(ConfigurableApplicationContext applicationContext) {
  applicationContext.refresh();
}
```
`createApplicationContext()`で生成したインスタンスのクラスはいずれも`AbstractApplicationContext`を継承していたが、その中で[refreshは定義されている](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L516-L579)。ここでようやくSpringBootからSpring本体のコードに入る

```java
// AbstractApplicationContext.java
public void refresh() throws BeansException, IllegalStateException {
  // 省略
  // Tell the subclass to refresh the internal bean factory.
  ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
  // 省略

  try {
    // Invoke factory processors registered as beans in the context.
    invokeBeanFactoryPostProcessors(beanFactory);
    // 省略
  }
  // 省略
}
```

[invokeBeanFactoryPostProcessors](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L706-L715)が呼び出される
```java
// AbstractApplicationContext.java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
  PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
  // 省略
}
```

[PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/support/PostProcessorRegistrationDelegate.java#L56-L187)が呼び出される
```java
// PostProcessorRegistrationDelegate.java
public static void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
  // 省略
  BeanDefinitionRegistryPostProcessor registryProcessor = (BeanDefinitionRegistryPostProcessor) postProcessor;
  registryProcessor.postProcessBeanDefinitionRegistry(registry);
  // 省略
}
```

[ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassPostProcessor.java#L223-L237)
```java
// ConfigurationClassPostProcessor.java
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
  // 省略
  processConfigBeanDefinitions(registry);
  // 省略
}
```
[processConfigBeanDefinitions](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassPostProcessor.java#L265-L366)
```java
// ConfigurationClassPostProcessor.java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
  List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
  String[] candidateNames = registry.getBeanDefinitionNames();
  for (String beanName : candidateNames) {
    // 省略
    configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
    // 省略
  }
  // 省略

  // Parse each @Configuration class
  ConfigurationClassParser parser = new ConfigurationClassParser(
    this.metadataReaderFactory, this.problemReporter, this.environment,
    this.resourceLoader, this.componentScanBeanNameGenerator, registry);
  Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
  // 省略
  parser.parse(candidates);
  // 省略
}
```
`@Configuration`の付与されたクラス(Configurationファイル)を`parser.parse(candidates);`でパースしている。`@Configuration`アノテーションは特に説明していなかったが、classに付与することでConfigurationファイルとして扱えるようにするためのアノテーションだ。`@SpringBootApplication` -> `@SpringBootConfiguration` -> `@Configuration`という継承関係になっている。
[ConfigurationClassParser.parse](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L169-L190)
```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
  for (BeanDefinitionHolder holder : configCandidates) {
    BeanDefinition bd = holder.getBeanDefinition();
    // 省略
    if (bd instanceof AnnotatedBeanDefinition) {
      parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
    }
    else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
      parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
    }
    else {
      parse(bd.getBeanClassName(), holder.getBeanName());
    }
    // 省略
  }
}
```
それぞれparseメソッドを呼び出している。これらの定義は[以下](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L195-L207)
```java
// ConfigurationClassParser.java
protected final void parse(@Nullable String className, String beanName) throws IOException {
	Assert.notNull(className, "No bean class name for configuration class bean definition");
	MetadataReader reader = this.metadataReaderFactory.getMetadataReader(className);
	processConfigurationClass(new ConfigurationClass(reader, beanName), DEFAULT_EXCLUSION_FILTER);
}

protected final void parse(Class<?> clazz, String beanName) throws IOException {
	processConfigurationClass(new ConfigurationClass(clazz, beanName), DEFAULT_EXCLUSION_FILTER);
}

protected final void parse(AnnotationMetadata metadata, String beanName) throws IOException {
	processConfigurationClass(new ConfigurationClass(metadata, beanName), DEFAULT_EXCLUSION_FILTER);
}
```
いずれにせよ、`processConfigurationClass`を呼び出している
[ConfigurationClassParser.processConfigurationClass](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L224-L254)
```java
// ConfigurationClassParser.java
protected void processConfigurationClass(ConfigurationClass configClass, Predicate<String> filter) throws IOException {
  // 省略
  SourceClass sourceClass = asSourceClass(configClass, filter);
  do{
    sourceClass = doProcessConfigurationClass(configClass, sourceClass, filter);
  }
  while (sourceClass != null);
  // 省略
}
```
[ConfigurationClassParser.doProcessConfigurationClass](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L256-L346)
```java
// ConfigurationClassParser.java
protected final SourceClass doProcessConfigurationClass(
  ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
  throws IOException {
  // 省略
  // Process any @ComponentScan annotations
  Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
    sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
  if (!componentScans.isEmpty() && !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
    for (AnnotationAttributes componentScan : componentScans) {
      // The config class is annotated with @ComponentScan -> perform the scan immediately
      Set<BeanDefinitionHolder> scannedBeanDefinitions = this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
      // 省略
    }
  }
  // 省略
}
```
`this.componentScanParser.parse`は`ComponentScanAnnotationParser.parse`である
[ComponentScanAnnotationParser.parse](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ComponentScanAnnotationParser.java#L76-L133)
```java
// ComponentScanAnnotationParser.java
public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, final String declaringClass) {
  // 省略
  return scanner.doScan(StringUtils.toStringArray(basePackages));
}
```
[ClassPathBeanDefinitionScanner.doScan](https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ClassPathBeanDefinitionScanner.java#L264-L297)
```java
// ClassPathBeanDefinitionScanner.java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
  // 省略
  for (String basePackage : basePackages) {
    Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
    for (BeanDefinition candidate : candidates) {
      ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
      candidate.setScope(scopeMetadata.getScopeName());
      // 省略
      if (candidate instanceof AnnotatedBeanDefinition) {
        AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
      }
      // 省略
      if (checkCandidate(beanName, candidate)) {
        BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
        definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
        beanDefinitions.add(definitionHolder);
        registerBeanDefinition(definitionHolder, this.registry);
      }
    }
  }
  // 省略
}
```
`registerBeanDefinition`でbean定義を登録していると思われる
