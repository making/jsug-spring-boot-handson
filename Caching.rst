キャッシュの利用
********************************************************************************


.. code-block:: console
    :emphasize-lines: 8,50,47,34,63-65

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



@Cacheableによるリザルトキャッシュ
================================================================================

.. code-block:: java
    :emphasize-lines: 4,18

    package jsug.domain.repository;

    import lombok.extern.slf4j.Slf4j;
    import org.springframework.cache.annotation.Cacheable;
    import org.springframework.core.io.ClassPathResource;
    import org.springframework.core.io.Resource;
    import org.springframework.stereotype.Component;
    import org.springframework.util.StreamUtils;

    import java.io.IOException;
    import java.io.InputStream;
    import java.nio.charset.StandardCharsets;

    @Component
    @Slf4j
    public class SqlFinder {

        @Cacheable("sql")
        public String get(String path) {
            Resource resource = new ClassPathResource(path);
            log.info("load {}", resource);
            try (InputStream stream = resource.getInputStream()) {
                return StreamUtils.copyToString(stream, StandardCharsets.UTF_8);
            } catch (IOException e) {
                throw new IllegalArgumentException(path + " is not found!", e);
            }
        }
    }


.. code-block:: java
    :emphasize-lines: 6,19

    package jsug.domain.service.category;

    import jsug.domain.model.Category;
    import jsug.domain.repository.category.CategoryRepository;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.cache.annotation.Cacheable;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import java.util.List;

    @Service
    @Transactional
    public class CategoryService {
        @Autowired
        CategoryRepository categoryRepository;

        @Transactional(readOnly = true)
        @Cacheable("category")
        public List<Category> findAll() {
            return categoryRepository.findAll();
        }
    }


.. code-block:: java
    :emphasize-lines: 6,21,28

    package jsug.domain.service.goods;

    import jsug.domain.model.Goods;
    import jsug.domain.repository.goods.GoodsRepository;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.cache.annotation.Cacheable;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;

    import java.util.UUID;

    @Service
    @Transactional
    public class GoodsService {
        @Autowired
        GoodsRepository goodsRepository;

        @Transactional(readOnly = true)
        @Cacheable("goods")
        public Goods findOne(UUID goodsId) {
            return goodsRepository.findOne(goodsId)
                    .orElseThrow(GoodsNotFoundException::new);
        }

        @Transactional(readOnly = true)
        @Cacheable("goods")
        public Page<Goods> findByCategoryId(int categoryId, Pageable pageable) {
            return goodsRepository.findByCategoryId(categoryId, pageable);
        }
    }



.. code-block:: java
    :emphasize-lines: 8-11,19,22,44-53

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
    import org.springframework.web.context.WebApplicationContext;

    import javax.sql.DataSource;
    import java.util.Arrays;

    @Configuration
    @EnableCaching
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
                    new ConcurrentMapCache("goods"),
                    new ConcurrentMapCache("sql")));
            return cacheManager;
        }
    }


カートの状態をキャッシュに保存
================================================================================


.. code-block:: java

    package jsug.infra.cart;

    import jsug.domain.model.Cart;
    import jsug.domain.model.OrderLine;
    import jsug.domain.model.OrderLines;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.cache.Cache;
    import org.springframework.cache.CacheManager;
    import org.springframework.context.ApplicationContext;
    import org.springframework.security.core.context.SecurityContextHolder;
    import org.springframework.web.context.request.RequestContextHolder;
    import org.springframework.web.context.request.ServletRequestAttributes;
    import org.springframework.web.servlet.support.RequestContextUtils;

    import java.util.ArrayList;
    import java.util.List;
    import java.util.Set;

    @Slf4j
    public class CachingCart extends Cart {
        public CachingCart() {
            super();
            loadCache();
        }

        @Override
        public OrderLines getOrderLines() {
            loadCache();
            return super.getOrderLines();
        }

        @Override
        public Cart add(OrderLine orderLine) {
            withSyncCache(() -> super.add(orderLine), true);
            return this;
        }

        @Override
        public Cart remove(Set<Integer> lineNo) {
            withSyncCache(() -> super.remove(lineNo), true);
            return this;
        }

        @Override
        public Cart clear() {
            withSyncCache(super::clear, true);
            return this;
        }

        Cache getCache() {
            // 本当はCacheManagerをDIしたかったが、
            // CacheManagerはSerializableじゃないで、SessionスコープであるCartのフィールドに入れられない
            // そのため、リクエストコンテキストから毎度ApplicationContextを取得して、
            // CacheManagerを取得している。
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            ApplicationContext context = RequestContextUtils.getWebApplicationContext(attributes.getRequest());
            CacheManager cacheManager = context.getBean(CacheManager.class);
            Cache cache = cacheManager.getCache("orderLines");
            return cache;
        }

        /**
         * OrderLinesをキャッシュと同期しながら処理を実行する
         *
         * @param action 主処理
         * @param save   処理後、キャッシュに保存するかどうか
         */
        void withSyncCache(Runnable action, boolean save) {
            Cache cache = getCache();
            String username = SecurityContextHolder.getContext().getAuthentication().getName();
            OrderLines orderLines = cache.get(username, OrderLines.class);
            if (orderLines != null) {
                // キャッシュから読み込み
                log.debug("load {} -> {}", username, orderLines);
                List<OrderLine> lines = new ArrayList<>(orderLines.getList()); // copy
                super.getOrderLines().getList().clear();
                super.getOrderLines().getList().addAll(lines);
            }
            // 処理
            action.run();
            if (save) {
                // キャッシュに保存
                if (log.isDebugEnabled()) {
                    log.debug("save {} -> {}", username, super.getOrderLines());
                }
                cache.put(username, super.getOrderLines());
            }
        }

        void loadCache() {
            withSyncCache(() -> {
            }, false);
        }
    }


.. code-block:: java
    :emphasize-lines: 4,42,51

    package jsug;

    import jsug.domain.model.Cart;
    import jsug.infra.cart.CachingCart;
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
    import org.springframework.web.context.WebApplicationContext;

    import javax.sql.DataSource;
    import java.util.Arrays;

    @Configuration
    @EnableCaching
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
        CacheManager cacheManager() {
            SimpleCacheManager cacheManager = new SimpleCacheManager();
            cacheManager.setCaches(Arrays.asList(
                    new ConcurrentMapCache("category"),
                    new ConcurrentMapCache("goods"),
                    new ConcurrentMapCache("orderLines"),
                    new ConcurrentMapCache("sql")));
            return cacheManager;
        }
    }