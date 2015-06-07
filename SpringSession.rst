Spring Sessionの導入
********************************************************************************

ここまで、作成したきたシステムは1サーバーで実行されることのみ考えられていました。

カートがHTTPセッション上で管理されているため、サーバーを複数台にスケールアウトした場合はセッションレプリケーションまたはロードバランサによって
同一セッションが同一サーバーに振り分けられるように設定する必要があります（スティッキーセッション）。

一般的に、セッションレプリケーションの設定はAPサーバー依存です。
また、スティッキーセッションは、サーバーがダウンするとそのサーバーが扱っていたセッション情報がなくなってしまうという課題もあります。

そこで登場したのが「Spring Session」です。Spring Sessionでは(Filterを経由することで)、\ ``HttpSession``\ の実装を上書きし、
\ ``HttpSession``\ のAPIを通じて透過的にRedisなどKVSにアクセスします。これにより、複数サーバーでセッション情報を共有することができます。

本章では、Spring SessionとRedisを使い、作成したアプリケーションを複数サーバー起動できるようにしましょう。


本章完了時点でのソースコードツリーを以下に示します。ハイライトされた部分が本章で追加・修正するファイルです。

.. code-block:: console
    :emphasize-lines: 2,8

    .
    ├── pom.xml
    └── src
        ├── main
        │   ├── java
        │   │   └── jsug
        │   │       ├── App.java
        │   │       ├── AppConfig.java
        │   │       ├── SecurityConfig.java
        │   │       ├── app
        │   │       │   ├── account
        │   │       │   │   ├── AccountController.java
        │   │       │   │   └── AccountForm.java
        │   │       │   ├── cart
        │   │       │   │   ├── CartController.java
        │   │       │   │   └── CartForm.java
        │   │       │   ├── goods
        │   │       │   │   ├── AddToCartForm.java
        │   │       │   │   └── GoodsController.java
        │   │       │   ├── login
        │   │       │   │   └── LoginController.java
        │   │       │   └── order
        │   │       │       └── OrderController.java
        │   │       ├── domain
        │   │       │   ├── model
        │   │       │   │   ├── Account.java
        │   │       │   │   ├── Cart.java
        │   │       │   │   ├── Category.java
        │   │       │   │   ├── Goods.java
        │   │       │   │   ├── Order.java
        │   │       │   │   ├── OrderLine.java
        │   │       │   │   └── OrderLines.java
        │   │       │   ├── repository
        │   │       │   │   ├── SqlFinder.java
        │   │       │   │   ├── account
        │   │       │   │   │   └── AccountRepository.java
        │   │       │   │   ├── category
        │   │       │   │   │   └── CategoryRepository.java
        │   │       │   │   ├── goods
        │   │       │   │   │   └── GoodsRepository.java
        │   │       │   │   └── order
        │   │       │   │       └── OrderRepository.java
        │   │       │   ├── service
        │   │       │   │   ├── account
        │   │       │   │   │   └── AccountService.java
        │   │       │   │   ├── category
        │   │       │   │   │   └── CategoryService.java
        │   │       │   │   ├── goods
        │   │       │   │   │   ├── GoodsNotFoundException.java
        │   │       │   │   │   └── GoodsService.java
        │   │       │   │   ├── order
        │   │       │   │   │   ├── EmptyCartOrderException.java
        │   │       │   │   │   ├── InvalidCartOrderException.java
        │   │       │   │   │   └── OrderService.java
        │   │       │   │   └── userdetails
        │   │       │   │       ├── ShopUserDetails.java
        │   │       │   │       └── ShopUserDetailsService.java
        │   │       │   └── validation
        │   │       │       ├── Confirm.java
        │   │       │       ├── ConfirmValidator.java
        │   │       │       ├── UnusedEmail.java
        │   │       │       └── UnusedEmailValidator.java
        │   │       └── infra
        │   │           ├── cart
        │   │           │   └── CachingCart.java
        │   │           └── logging
        │   │               └── HandlerExceptionResolverLoggingAspect.java
        │   └── resources
        │       ├── application.properties
        │       ├── db
        │       │   └── migration
        │       │       ├── V1__create-schema.sql
        │       │       └── V2__initial-data.sql
        │       ├── log4jdbc.log4j2.properties
        │       ├── messages.properties
        │       ├── sql
        │       │   ├── account
        │       │   │   ├── countByEmail.sql
        │       │   │   ├── create.sql
        │       │   │   └── findOne.sql
        │       │   ├── category
        │       │   │   └── findAll.sql
        │       │   ├── goods
        │       │   │   ├── countByCategoryId.sql
        │       │   │   ├── findByCategoryId.sql
        │       │   │   └── findOne.sql
        │       │   ├── order
        │       │   │   └── create.sql
        │       │   └── orderLine
        │       │       └── create.sql
        │       ├── static
        │       │   ├── css
        │       │   │   └── wro.css
        │       │   ├── fonts
        │       │   │   ├── montserrat-webfont.eot
        │       │   │   ├── montserrat-webfont.svg
        │       │   │   ├── montserrat-webfont.ttf
        │       │   │   ├── montserrat-webfont.woff
        │       │   │   ├── varela_round-webfont.eot
        │       │   │   ├── varela_round-webfont.svg
        │       │   │   ├── varela_round-webfont.ttf
        │       │   │   └── varela_round-webfont.woff
        │       │   └── images
        │       │       ├── 404-icon.png
        │       │       ├── homepage-bg.jpg
        │       │       ├── platform-bg.png
        │       │       ├── platform-spring-xd.png
        │       │       ├── spring-logo-xd-mobile.png
        │       │       └── spring-logo-xd.png
        │       └── templates
        │           ├── account
        │           │   ├── createFinish.html
        │           │   └── createForm.html
        │           ├── cart
        │           │   └── viewCart.html
        │           ├── error.html
        │           ├── goods
        │           │   ├── notFound.html
        │           │   └── showGoods.html
        │           ├── login
        │           │   └── loginForm.html
        │           └── order
        │               ├── confirm.html
        │               ├── error.html
        │               └── finish.html
        └── test
            ├── java
            │   └── jsug
            │       └── domain
            │           ├── TestConfig.java
            │           ├── model
            │           │   └── CartTest.java
            │           ├── repository
            │           │   ├── account
            │           │   │   └── AccountRepositoryTest.java
            │           │   ├── category
            │           │   │   └── CategoryRepositoryTest.java
            │           │   ├── goods
            │           │   │   └── GoodsRepositoryTest.java
            │           │   └── order
            │           │       └── OrderRepositoryTest.java
            │           └── service
            │               ├── account
            │               │   └── AccountServiceTest.java
            │               ├── goods
            │               │   └── GoodsServiceTest.java
            │               ├── order
            │               │   └── OrderServiceTest.java
            │               └── userdetails
            │                   └── ShopUserDetailsServiceTest.java
            └── resources
                ├── logback.xml
                └── sql
                    ├── drop-tables.sql
                    ├── insert-accounts.sql
                    ├── insert-category.sql
                    ├── insert-goods.sql
                    └── insert-orders.sql

pom.xmlに以下の依存関係を追加してください。

.. code-block:: xml


    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session</artifactId>
        <version>1.0.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-redis</artifactId>
    </dependency>

Spring BootでSpring Sessionを使うための設定はとても簡単で、以下のように、JavaConfigに\ ``@EnableRedisHttpSession``\ をつけるだけです。

.. code-block:: java
    :emphasize-lines: 16,24

    package jsug;

    import jsug.domain.model.Cart;
    import net.sf.log4jdbc.sql.jdbcapi.DataSourceSpy;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
    import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
    import org.springframework.cache.CacheManager;
    import org.springframework.cache.annotation.EnableCaching;
    import org.springframework.cache.concurrent.ConcurrentMapCache;
    import org.springframework.cache.support.SimpleCacheManager;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Scope;
    import org.springframework.context.annotation.ScopedProxyMode;
    import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;
    import org.springframework.web.context.WebApplicationContext;

    import javax.sql.DataSource;
    import java.util.Arrays;

    @Configuration
    @EnableCaching
    @EnableRedisHttpSession
    public class AppConfig {
        @Autowired
        DataSourceProperties dataSourceProperties;

        @Bean
        DataSource dataSource() {
            DataSource dataSource = DataSourceBuilder
                    .create(this.dataSourceProperties.getClassLoader())
                    .url(this.dataSourceProperties.getUrl())
                    .username(this.dataSourceProperties.getUsername())
                    .password(this.dataSourceProperties.getPassword())
                    .build();
            return new DataSourceSpy(dataSource);
        }

        @Bean
        @Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
        Cart cart() {
            return new Cart();
        }

        @Bean
        CacheManager cacheManager() {
            SimpleCacheManager cacheManager = new SimpleCacheManager();
            cacheManager.setCaches(Arrays.asList(
                    new ConcurrentMapCache("category"),
                    new ConcurrentMapCache("goods"),push
                    new ConcurrentMapCache("sql")));
            return cacheManager;
        }
    }

Spring Sessionとは直接関係ありませんが、\ ``CacheManager``\ 実装もSpring Data Redisが提供している\ ``RedisCacheManager``\ に差し替えます。
これにより、カートの情報やリザルトキャッシュもサーバー間で共有できます。

.. code-block:: java
    :emphasize-lines: 7, 16-19,49-50

    package jsug;

    import jsug.domain.model.Cart;
    import jsug.infra.cart.CachingCart;
    import net.sf.log4jdbc.sql.jdbcapi.DataSourceSpy;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
    import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
    import org.springframework.cache.CacheManager;
    import org.springframework.cache.annotation.EnableCaching;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Scope;
    import org.springframework.context.annotation.ScopedProxyMode;
    import org.springframework.data.redis.cache.RedisCacheManager;
    import org.springframework.data.redis.core.RedisOperations;
    import org.springframework.data.redis.core.RedisTemplate;
    import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;
    import org.springframework.web.context.WebApplicationContext;

    import javax.sql.DataSource;

    @Configuration
    @EnableCaching
    @EnableRedisHttpSession
    public class AppConfig {
        @Autowired
        DataSourceProperties dataSourceProperties;

        @Bean
        DataSource dataSource() {
            DataSource dataSource = DataSourceBuilder
                    .create(this.dataSourceProperties.getClassLoader())
                    .url(this.dataSourceProperties.getUrl())
                    .username(this.dataSourceProperties.getUsername())
                    .password(this.dataSourceProperties.getPassword())
                    .build();
            return new DataSourceSpy(dataSource);
        }

        @Bean
        @Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
        Cart cart() {
            return new CachingCart();
        }

        @Bean
        CacheManager cacheManager(@Qualifier("redisTemplate") /* (1) */ RedisOperations<Object, Object> redisOperations) {
            return new RedisCacheManager((RedisTemplate) redisOperations);
        }

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | Spring Data Redisを依存関係に追加すると、Spring BootのAutoConfigureにより、2種類の\ ``RedisOperations``\ がDIコンテナに登録されます。
       | \ ``RedisCacheManager``\ のコンストラクタで必要なのは\ ``RedisOperations``\ の実装クラスの\ ``RedisTemplate``\ ですので、\ ``@Qualifier``\ でBean名を指定してインジェクションします。
       | ここはSpring Boot 1.3で改善されると思います。


それでは複数サーバーを立ち上げましょう。Redisサーバーも起動してください。

.. code-block:: console

    $ mvn clean package # jar作成
    $ cd target
    $ java -jar jsug-shop-1.0-SNAPSHOT.jar --server.port=8080 # 1台目
    $ java -jar jsug-shop-1.0-SNAPSHOT.jar --server.port=8080 # 2台目
    $ java -jar jsug-shop-1.0-SNAPSHOT.jar --server.port=8080 # 3台目

8080-8082番ポートどれにアクセスしても、カートの情報が共有されていることを確認してください。

Redisのホスト名、ポートがデフォルt(localhost, 6379)でない場合は、以下のように明示してください

.. code-block:: console

    $ java -jar jsug-shop-1.0-SNAPSHOT.jar --server.port=8080 --spring.redis.host=192.168.99.100 --spring.redis.port=6379 # 1台目
    $ java -jar jsug-shop-1.0-SNAPSHOT.jar --server.port=8081 --spring.redis.host=192.168.99.100 --spring.redis.port=6379 # 2台目
    $ java -jar jsug-shop-1.0-SNAPSHOT.jar --server.port=8082 --spring.redis.host=192.168.99.100 --spring.redis.port=6379 # 3台目