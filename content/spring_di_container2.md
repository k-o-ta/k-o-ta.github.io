+++
title = "DI contaier in spring"
date = 2020-08-18
draft = true
[taxonomies]
tags = []
+++

# spring boot ソースコードリーディング(コンポーネントスキャン)
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

利用例
```java
// DIコンテナからインスタンスを取得する例
ApplicationContext = new AnnotationConfigAppricationContext(AppConfig.class);
MyService myService = context.getBean(MyService.class);
```

```
```


1. DIコンテナへbeanの登録を定義する(Configuration)
beanの定義方法(beanの登録方法)
* Java Config
* XML Config
* アノテーションベースConfig
bean定義の例
* JavaConfigの例
```java
@Configuration // configファイルであることの宣言
@ComponentScan("com.myexmple.app") // 後述のコンポーネントスキャンをJavaConfigからも行える
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
* XMLの例
省略

* アノテーションベースの例
  * configurationファイルではなく、コンポーネントの実装ファイルにconfig用アノテーションを書くとコンポーネントスキャンされてDIコンテナに登録される
```java
package com.myexample.app

@Component // @Componentアノテーションを付けるとコンポーネントスキャンの対象になる
public class MyServiceImpl implements MyService {
  @Autowired // 型が一致するBeanがDIコンテナにあったら注入する
  public MyServiceImpl(MyRepository myRepository) {}
}
```

2. ApplicationContextインスタンスの生成
Configurationの方法によって生成方法が違う
```java
// Java Configを使った生成. @Configurationが付いていたらConfigファイルとして利用する
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
// XM Configを使った生成
ApplicationContext applicationContext = new FileSystemXmlApplicationContext("./applicationContext.xml");
// アノテーションベースConfigを使った生成. package配下をコンポーネントスキャンする
ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.myexample.app")
```

3. DIコンテナからbeanを取得する
ルックアップの方法
* 取得するBeanの名前を指定する(`applicationContext.getBean("myService");`)
  * Object型で取得されるので嬉しくない
* 取得するBeanの型を指定する(`applicationContext.getBean(MyService.class);`)
  * 指定する型のBeanが一つだけのとき使える
* 取得するBeanの名前と型を指定する(`applicationContext.getBean("myService", MyService.class);`)
  * 指定する型のBeanが複数存在する場合に使う

Spring bootのAutoConfigure
beanの定義とかが不要になる。
@SpringBootApplicationをつけると、@Configurationと@EnableAutoConfigurationと@ComponentScanの組み合わせが付与され、そのクラスがconfiguratoinクラスになり、@BeanアノテーションでBean定義でき、クラス配下のパッケージがコンポーネントスキャン対象になる。

@ComponentScanの実装を読む

Spring bootじゃないとき
`ApplicationContext applicationContext = new AnnotationConfigApplicationContext("com.myexample.app")
`を呼ぶ場合,
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/AnnotationConfigApplicationContext.java#L100
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ClassPathBeanDefinitionScanner.java#L251


Spring bootだったり、@ComponentScanを使っていたりするとき
```java
SpringApplication.run
  ConfigurableApplicationContext context = null;
  context = this.createApplicationContext();
  this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
  this.refreshContext(context);
    this.refresh(context);
      Assert.isInstanceOf(AbstractApplicationContext.class, applicationContext);
      ((AbstractApplicationContext)applicationContext).refresh();
```

```java
AbstractApplicationContext#refresh
  AbstractApplicationContext#invokeBeanFactoryPostProcessors
AbstractApplicationContext#invokeBeanFactoryPostProcessors
  PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors
PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors
  invokeBeanFactoryPostProcessors
    postProcessBeanFactory
```

postProcessBeanDefinitionRegistry
postProcessBeanFactory


https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassPostProcessor.java#L265
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassPostProcessor.java#L319

https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L169
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L195-L207
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L224
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L249
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L265-L266
// Process any @ComponentScan annotations
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L287-L289
// The config class is annotated with @ComponentScan -> perform the scan immediately
```java
Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
				sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
```

https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java#L293-L295
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ComponentScanAnnotationParser.java#L132
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ClassPathBeanDefinitionScanner.java#L276
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ClassPathBeanDefinitionScanner.java#L285
https://github.com/spring-projects/spring-framework/blob/v5.2.8.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ClassPathBeanDefinitionScanner.java#L289-L292
registerBeanDefinitionと返り値のbeanDefinitionsのどちらが重要なのか?



autoconfigureの実装を読む

