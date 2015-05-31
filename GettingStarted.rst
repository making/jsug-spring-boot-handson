プロジェクトのセットアップ
********************************************************************************

本章ではプロジェクトをセットアップし、プロジェクトの内容を確認します。
コードは書きません。

ハンズオンプロジェクトの使い方
================================================================================

まずは、ベースとなるハンズオンプロジェクトをGitで取得します。

.. code-block:: console

    $ git clone https://github.com/making/jsug-shop

このプロジェクトには各章ごとの完成コードが含まれています。

.. code-block:: console

    $ git branch
    * ch01
      ch02
      ch03
      ch04
      ch05

\ ``ch01``\ ブランチになっていない場合は、以下のコマンドでブランチを切り替えてください。

.. code-block:: console

    $ git checkout ch01

ハンズオン用に作業ブランチを作成しましょう。

.. code-block:: console

    $ git checkout -b handson

ハンズオンは基本的にこのブランチで進めます。

もし、途中の章の完成コードからハンズオンを再開したい場合は、以下のように途中のブランチから
ハンズオンブランチを作成してください。

.. code-block:: console

    $ git checkout -f ch02 # 2章完了後から始める場合
    $ git branch -D handson
    $ git checkout -b handson


ハンズオンプロジェクトの確認
================================================================================

ハンズオンプロジェクトのソースコードはある程度作成済みです。
構成を確認しましょう。

.. code-block:: console

    $ tree
    .
    ├── pom.xml
    └── src
        ├── main
        │   ├── java
        │   │   └── jsug
        │   │       ├── App.java ... Spring Bootアプリケーションのエンドポイント
        │   │       ├── AppConfig.java ... 今回のアプリケーション用のBean定義ファイル
        │   │       ├── SecurityConfig.java ... 認証・認可(Spring Security)用のBean定義ファイル
        │   │       └── domain
        │   │           ├── model ... ドメインオブジェクトを格納するパッケージ
        │   │           │   ├── Account.java ... アカウントモデル
        │   │           │   ├── Cart.java ... カートモデル
        │   │           │   ├── Category.java ... カテゴリーモデル
        │   │           │   ├── Goods.java ... 商品モデル
        │   │           │   ├── Order.java ... 注文モデル
        │   │           │   ├── OrderLine.java ... 注文行(1行)モデル
        │   │           │   └── OrderLines.java ... 注文行(複数)モデル
        │   │           ├── repository ... レポジトリクラスを格納するパッケージ
        │   │           │   ├── SqlFinder.java ... SQLファイルから、SQL文を取得するためのヘルパー
        │   │           │   ├── account
        │   │           │   │   └── AccountRepository.java ... アカウントモデルのデータアクセス(CRUD)用クラス
        │   │           │   ├── category
        │   │           │   │   └── CategoryRepository.java ... カテゴリモデルのデータアクセス(CRUD)用クラス
        │   │           │   ├── goods
        │   │           │   │   └── GoodsRepository.java ... 商品モデルのデータアクセス(CRUD)用クラス
        │   │           │   └── order
        │   │           │       └── OrderRepository.java ... 注文モデルのデータアクセス(CRUD)用クラス
        │   │           ├── service ... サービスクラスを格納するパッケージ
        │   │           │   ├── account
        │   │           │   │   └── AccountService.java ... アカウントのサービスクラス
        │   │           │   ├── category
        │   │           │   │   └── CategoryService.java ... カテゴリのサービスクラス
        │   │           │   ├── goods
        │   │           │   │   ├── GoodsNotFoundException.java ... 商品が見つからない場合の例外クラス
        │   │           │   │   └── GoodsService.java ... 商品のサービスクラス
        │   │           │   ├── order
        │   │           │   │   ├── EmptyCartOrderException.java ... カートが空の場合の例外クラス
        │   │           │   │   ├── InvalidCartOrderException.java ... カートの状態が不正な場合の例外クラス
        │   │           │   │   └── OrderService.java ... 注文のサービスクラス
        │   │           │   └── userdetails
        │   │           │       ├── ShopUserDetails.java ... 認証ユーザークラス
        │   │           │       └── ShopUserDetailsService.java ... 認証ユーザー取得サービスクラス
        │   │           └── validation ... カスタムBeanValidationルールを格納するパッケージ
        │   │               ├── Confirm.java ... 2つのフィールドが同じ値であることの制約を示すアノテーション
        │   │               ├── ConfirmValidator.java ... 2つのフィールドが同じ値であることを検証するバリデータ
        │   │               ├── UnusedEmail.java ... E-mailが使用されていないことの制約を示すアノテーション
        │   │               └── UnusedEmailValidator.java ... E-mailが使用されていないことを検証するバリデータ
        │   └── resources
        │       ├── application.properties ... アプリケーションの設定ファイル
        │       ├── db ... FlywayによるDBマイグレーション用のSQLファイル
        │       │   └── migration
        │       │       ├── V1__create-schema.sql
        │       │       └── V2__initial-data.sql
        │       ├── log4jdbc.log4j2.properties ... Log4JDBCの設定ファイル
        │       ├── sql ... リポジトリ用のSQLファイル
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
        │       ├── static ... 静的ファイル
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
        │       └── templates ... Thyemeleafのテンプレート
        │           ├── account
        │           │   ├── createFinish.html ... アカウント作成完了画面
        │           │   └── createForm.html ... アカウント作成フォーム画面
        │           ├── cart
        │           │   └── viewCart.html ... カート確認画面
        │           ├── goods
        │           │   ├── notFound.html ... 商品が見つからない場合のエラー画面
        │           │   └── showGoods.html ... 商品一覧画面
        │           ├── login
        │           │   └── loginForm.html ... ログインフォーム画面
        │           └── order
        │               ├── confirm.html ... 注文確認画面
        │               ├── error.html ... 注文エラー画面
        │               └── finish.html ... 注文確認画面
        └── test
            ├── java
            │   └── jsug
            │       └── domain
            │           ├── TestConfig.java ... ユニットテスト用のBean定義ファイル
            │           ├── model ... ドメインオブジェクトのJUnitコードを格納するパッケージ
            │           │   └── CartTest.java
            │           ├── repository ... レポジトリクラスのJUnitコードを格納するパッケージ
            │           │   ├── account
            │           │   │   └── AccountRepositoryTest.java
            │           │   ├── category
            │           │   │   └── CategoryRepositoryTest.java
            │           │   ├── goods
            │           │   │   └── GoodsRepositoryTest.java
            │           │   └── order
            │           │       └── OrderRepositoryTest.java
            │           └── service ... サービスクラスのJUnitコードを格納するパッケージ
            │               ├── account
            │               │   └── AccountServiceTest.java
            │               ├── goods
            │               │   └── GoodsServiceTest.java
            │               ├── order
            │               │   └── OrderServiceTest.java
            │               └── userdetails
            │                   └── ShopUserDetailsServiceTest.java
            └── resources
                ├── logback.xml ... ユニットテスト用Logback設定ファイル
                └── sql ... テストデータ作成用のSQLファイル
                    ├── drop-tables.sql
                    ├── insert-accounts.sql
                    ├── insert-category.sql
                    ├── insert-goods.sql
                    └── insert-orders.sql


あらかじめ、データアクセスコード、業務ロジック、画面は作成済みです。
ハンズオンでは主にコントローラとインフラストラクチャーコードを作成します。
作成済みのコードに関しては「はじめてのSpring Boot」で扱っている内容と重複するため、本資料では特に説明しません。

なお、アプリケーションを簡単にするため、今回作るショッピングサイトには「在庫」の概念はありません。


JUnitの実行
================================================================================

今回のハンズオンプロジェクトでは、作成済みのコード(リポジトリクラス、サービスクラス)に対して、JUnitコードが用意されています。
ハンズオンを進める前に、JUnitを実行して動作確認しておきましょう。

.. code-block:: console

    $ mvn test
    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------------------------------------------------------------
    [INFO] Building JSUG Shop 1.0-SNAPSHOT
    [INFO] ------------------------------------------------------------------------
    [INFO]
    [INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ jsug-shop ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] Copying 1 resource
    [INFO] Copying 36 resources
    [INFO]
    [INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ jsug-shop ---
    [INFO] Changes detected - recompiling the module!
    [INFO] Compiling 28 source files to /Users/maki/git/jsug-shop/target/classes
    [INFO]
    [INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ jsug-shop ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] Copying 6 resources
    [INFO]
    [INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ jsug-shop ---
    [INFO] Changes detected - recompiling the module!
    [INFO] Compiling 10 source files to /Users/maki/git/jsug-shop/target/test-classes
    [INFO]
    [INFO] --- maven-surefire-plugin:2.17:test (default-test) @ jsug-shop ---
    [INFO] Surefire report directory: /Users/maki/git/jsug-shop/target/surefire-reports

    -------------------------------------------------------
     T E S T S
    -------------------------------------------------------
    Running jsug.domain.model.CartTest
    Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.22 sec - in jsug.domain.model.CartTest
    Running jsug.domain.repository.account.AccountRepositoryTest
    2015-05-31 03:11:34.040  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/countByEmail.sql]
    2015-05-31 03:11:34.114  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/findOne.sql]
    2015-05-31 03:11:34.140  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/create.sql]
    2015-05-31 03:11:34.142  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/findOne.sql]
    2015-05-31 03:11:34.161  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/findOne.sql]
    2015-05-31 03:11:34.185  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/countByEmail.sql]
    Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.897 sec - in jsug.domain.repository.account.AccountRepositoryTest
    Running jsug.domain.repository.category.CategoryRepositoryTest
    2015-05-31 03:11:34.227  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/category/findAll.sql]
    Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.044 sec - in jsug.domain.repository.category.CategoryRepositoryTest
    Running jsug.domain.repository.goods.GoodsRepositoryTest
    2015-05-31 03:11:34.277  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/goods/findOne.sql]
    2015-05-31 03:11:34.317  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/goods/findOne.sql]
    2015-05-31 03:11:34.339  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/goods/findByCategoryId.sql]
    2015-05-31 03:11:34.344  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/goods/countByCategoryId.sql]
    2015-05-31 03:11:34.363  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/goods/findByCategoryId.sql]
    2015-05-31 03:11:34.379  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/goods/countByCategoryId.sql]
    Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.14 sec - in jsug.domain.repository.goods.GoodsRepositoryTest
    Running jsug.domain.repository.order.OrderRepositoryTest
    2015-05-31 03:11:34.411  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/order/create.sql]
    2015-05-31 03:11:34.413  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/orderLine/create.sql]
    Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.045 sec - in jsug.domain.repository.order.OrderRepositoryTest
    Running jsug.domain.service.account.AccountServiceTest
    2015-05-31 03:11:34.455  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/countByEmail.sql]
    2015-05-31 03:11:34.473  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/create.sql]
    2015-05-31 03:11:34.475  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/findOne.sql]
    2015-05-31 03:11:34.490  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/countByEmail.sql]
    Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.057 sec - in jsug.domain.service.account.AccountServiceTest
    Running jsug.domain.service.goods.GoodsServiceTest
    2015-05-31 03:11:34.552  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/goods/findOne.sql]
    2015-05-31 03:11:34.572  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/goods/findOne.sql]
    Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.086 sec - in jsug.domain.service.goods.GoodsServiceTest
    Running jsug.domain.service.order.OrderServiceTest
    2015-05-31 03:11:34.637  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/order/create.sql]
    2015-05-31 03:11:34.639  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/orderLine/create.sql]
    Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.099 sec - in jsug.domain.service.order.OrderServiceTest
    Running jsug.domain.service.userdetails.ShopUserDetailsServiceTest
    2015-05-31 03:11:34.713  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/findOne.sql]
    2015-05-31 03:11:34.731  INFO   --- [           main] jsug.domain.repository.SqlFinder         : load class path resource [sql/account/findOne.sql]
    Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.041 sec - in jsug.domain.service.userdetails.ShopUserDetailsServiceTest

    Results :

    Tests run: 22, Failures: 0, Errors: 0, Skipped: 0

    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 9.323 s
    [INFO] Finished at: 2015-05-31T03:11:34+09:00
    [INFO] Final Memory: 26M/280M
    [INFO] ------------------------------------------------------------------------

22件のテストが全て成功していれば、動作確認完了です。


JUnitコードの確認
================================================================================

先ほど実行したJUnitのコードを見てみましょう。ここでは\ ``AccountServiceTest``\ を見ます。

.. code-block:: java

    package jsug.domain.service.account;

    import jsug.domain.TestConfig;
    import jsug.domain.model.Account;
    import jsug.domain.repository.account.AccountRepository;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.test.context.ContextConfiguration;
    import org.springframework.test.context.jdbc.Sql;
    import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

    import java.time.LocalDate;

    import static org.junit.Assert.*;
    import static org.hamcrest.CoreMatchers.*;

    @RunWith(SpringJUnit4ClassRunner.class) // (1)
    @ContextConfiguration(classes = TestConfig.class) // (2)
    @Sql(scripts = { // (3)
            "/sql/drop-tables.sql",
            "/db/migration/V1__create-schema.sql",
            "/sql/insert-accounts.sql"
    })
    public class AccountServiceTest {
        @Autowired
        AccountService accountService; // (4)
        @Autowired
        AccountRepository accountRepository;

        @Test
        public void testIsUnusedEmail_Used() throws Exception {
            assertThat(accountService.isUnusedEmail("demo1@example.com"), is(false));
        }

        @Test
        public void testIsUnusedEmail_Unused() throws Exception {
            assertThat(accountService.isUnusedEmail("hoge@example.com"), is(true));
        }

        @Test
        public void testRegister() throws Exception {
            Account account = Account.builder()
                    .email("foo@example.com")
                    .name("Taro Yamada")
                    .birthDay(LocalDate.of(2000, 1, 1))
                    .zip("1000000")
                    .address("Tokyo")
                    .build();
            Account created = accountService.register(account, "password");
            assertThat(created.getPassword(), is("password"));
            Account found = accountRepository.findOne("foo@example.com").get();
            assertThat(found, is(created));
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | JUnitでSpringのDIコンテナを使うためのRunnerを指定します。
   * - | (2)
     - | \ ``@ContextConfiguration``\ でテスト用のBean定義ファイルを指定します。XMLでもJavaConfigでも指定できます。
       | 省略した場合は、同じパッケージの\ ``クラス名-context.xml``\ が読み込まれます。またはJUnitクラス内にインナークラスのJavaConfigがあれば、そちらを使います。
   * - | (3)
     - | \ ``@Sql``\ でユニットテスト内で実行させるSQLのパスを指定します。この機能はSpring 4.1から追加された機能です。
       | ここでは、組み込みデータベース向けに、DDL実行、テストデータ投入を行っています。デフォルトでは毎テストケースの開始時に実行されます。
       | テストの順序に依存しないように、毎回データベースを初期化するのが良いです。
   * - | (4)
     - | JUnitコードにDIコンテナ管理のBeanをインジェクションできます。

次に、ユニットテスト用のBean定義ファイルである、\ ``TestConfig``\ を見てみます。

.. code-block:: java

    package jsug.domain;


    import net.sf.log4jdbc.sql.jdbcapi.DataSourceSpy;
    import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.ComponentScan;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
    import org.springframework.jdbc.datasource.DataSourceTransactionManager;
    import org.springframework.security.crypto.password.NoOpPasswordEncoder;
    import org.springframework.security.crypto.password.PasswordEncoder;
    import org.springframework.transaction.PlatformTransactionManager;
    import org.springframework.transaction.annotation.EnableTransactionManagement;

    import javax.sql.DataSource;

    @Configuration
    @EnableTransactionManagement // (1)
    @ComponentScan({ // (2)
            "jsug.domain.repository",
            "jsug.domain.service"})
    public class TestConfig {
        @Bean // (3)
        DataSource dataSource() {
            DataSource dataSource = DataSourceBuilder.create()
                    .url("jdbc:h2:mem:jsug-shop;DB_CLOSE_ON_EXIT=FALSE")
                    .build();
            return new DataSourceSpy(dataSource);
        }

        @Bean // (4)
        NamedParameterJdbcTemplate jdbcTemplate() {
            return new NamedParameterJdbcTemplate(dataSource());
        }

        @Bean // (5)
        PlatformTransactionManager transactionManager() {
            return new DataSourceTransactionManager(dataSource());
        }

        @Bean // (6)
        PasswordEncoder passwordEncoder() {
            return NoOpPasswordEncoder.getInstance();
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | トランザクション管理を有効にします。
   * - | (2)
     - | テスト用にコンポーネントスキャンするパッケージを指定します。ここではレポジトリクラスとサービスクラスだけを対象にしています。
   * - | (3)
     - | テスト用のデータソースを設定します。
   * - | (4)
     - | テスト用の\ ``NamedParameterJdbcTemplate``\ を設定します。
   * - | (5)
     - | テスト用のトランザクションマネージャを設定します。
   * - | (6)
     - | テスト用の\ ``PasswordEncoder``\ を設定します。ここでは、生パスワードをエンコードせず、そのまま使う\ ``PasswordEncoder``\ を使用します。

ここで紹介したテスト機能はSpring Bootとは関係なく、Spring Frameworkの機能です。詳しくは\ `リファレンスマニュアル <http://docs.spring.io/spring/docs/current/spring-framework-reference/html/testing.html>`__\ を確認してください。
