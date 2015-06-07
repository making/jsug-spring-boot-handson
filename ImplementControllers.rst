Controllerの実装
********************************************************************************

\ ``git clone``\ したプロジェクトにはWebアプリケーションとしての機能が実装されていません。

Spring MVCのControllerを実装して、Webアプリケーションを作成しましょう。

以下に実装する画面の情報を示します。

.. list-table::
   :header-rows: 1

   * - 処理名
     - HTTPメソッド
     - パス
     - 画面名
   * - | ログインフォーム画面表示処理
     - | GET
     - | /loginForm
     - | login/loginForm
   * - | アカウント作成フォーム画面表示処理
     - | GET
     - | /account/create?form
     - | account/createForm
   * - | アカウント作成処理
     - | POST
     - | /account/create
     - | アカウント作成完了画面表示処理へリダイレクト
   * - | アカウント作成完了画面表示処理
     - | GET
     - | /account/create?finish
     - | account/createFinish
   * - | 商品一覧画面表示処理
     - | GET
     - | /
     - | goods/showGoods
   * - | 商品一覧画面表示(カテゴリ指定)処理
     - | GET
     - | /?categoryId={categoryId}
     - | goods/showGoods
   * - | 商品をカートへ追加処理
     - | POST
     - | /addToCart
     - | カート表示画面処理へリダイレクト
   * - | カート表示画面処理
     - | GET
     - | /cart
     - | cart/viewCart
   * - | 商品をカートから削除処理
     - | POST
     - | /cart
     - | カート表示画面処理へリダイレクト
   * - | 注文確認画面処理
     - | GET
     - | /order?confirm
     - | order/confirm
   * - | 注文処理
     - | POST
     - | /order
     - | 注文完了画面表示処理へリダイレクト
   * - | 注文完了画面表示処理
     - | GET
     - | /order?finish
     - | order/finish

本章完了時点でのソースコードツリーを以下に示します。ハイライトされた部分が本章で追加・修正するファイルです。

.. code-block:: console
  :emphasize-lines: 9,11-24,71

  $ tree .
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
      │   │       └── domain
      │   │           ├── model
      │   │           │   ├── Account.java
      │   │           │   ├── Cart.java
      │   │           │   ├── Category.java
      │   │           │   ├── Goods.java
      │   │           │   ├── Order.java
      │   │           │   ├── OrderLine.java
      │   │           │   └── OrderLines.java
      │   │           ├── repository
      │   │           │   ├── SqlFinder.java
      │   │           │   ├── account
      │   │           │   │   └── AccountRepository.java
      │   │           │   ├── category
      │   │           │   │   └── CategoryRepository.java
      │   │           │   ├── goods
      │   │           │   │   └── GoodsRepository.java
      │   │           │   └── order
      │   │           │       └── OrderRepository.java
      │   │           ├── service
      │   │           │   ├── account
      │   │           │   │   └── AccountService.java
      │   │           │   ├── category
      │   │           │   │   └── CategoryService.java
      │   │           │   ├── goods
      │   │           │   │   ├── GoodsNotFoundException.java
      │   │           │   │   └── GoodsService.java
      │   │           │   ├── order
      │   │           │   │   ├── EmptyCartOrderException.java
      │   │           │   │   ├── InvalidCartOrderException.java
      │   │           │   │   └── OrderService.java
      │   │           │   └── userdetails
      │   │           │       ├── ShopUserDetails.java
      │   │           │       └── ShopUserDetailsService.java
      │   │           └── validation
      │   │               ├── Confirm.java
      │   │               ├── ConfirmValidator.java
      │   │               ├── UnusedEmail.java
      │   │               └── UnusedEmailValidator.java
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



LoginControllerの実装
================================================================================

まずはログイン画面を返す簡単なControllerに以下の処理を実装します。

* ログインフォーム画面表示処理

\ ``src/main/java/jsug/app/login/LoginController.java``\ を作成して、以下のコードを記述してください。

.. code-block:: java

  package jsug.app.login;

  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;

  @Controller
  public class LoginController {
      @RequestMapping("loginForm")
      String loginForm() {
          return "login/loginForm";
      }
  }

特筆すべき内容はありません。\ ``/loginForm``\ にアクセスすると、\ ``classpath:/templates/login/loginForm.html``\ がレンダリングされます。
HTMLは既に準備されています。Controllerを実装したら、\ ``App.java``\ を実行してアプリケーションを起動してください。

http://localhost:8080\ にアクセスすると、認可制御により\ http://localhost:8080/loginForm\ に遷移します。

AccountControllerの実装
================================================================================

次は\ ``AccountController``\ に以下の処理を実装します。

* アカウント作成フォーム画面表示処理
* アカウント作成処理
* アカウント作成完了画面表示処理


\ ``src/main/java/jsug/app/account/AccountController.java``\ を作成して、以下のコードを記述してください。


.. code-block:: java

  package jsug.app.account;

  import jsug.domain.model.Account;
  import jsug.domain.service.account.AccountService;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.stereotype.Controller;
  import org.springframework.validation.BindingResult;
  import org.springframework.validation.annotation.Validated;
  import org.springframework.web.bind.annotation.ModelAttribute;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RequestMethod;
  import org.springframework.web.servlet.mvc.support.RedirectAttributes;

  @Controller
  @RequestMapping("account")
  public class AccountController {
      @Autowired // (1)
      AccountService accountService;

      @ModelAttribute // (2)
      AccountForm setupForm() {
          return new AccountForm();
      }

      @RequestMapping(value = "create", params = "form", method = RequestMethod.GET) // (3)
      String createForm() {
          return "account/createForm";
      }

      @RequestMapping(value = "create", method = RequestMethod.POST) // (4)
      String create(@Validated AccountForm form /* (5) */, BindingResult bindingResult,
                    RedirectAttributes attributes) {
          if (bindingResult.hasErrors()) { // (6)
              return "account/createForm";
          }
          Account account = Account.builder() // (7)
                  .name(form.getName())
                  .email(form.getEmail())
                  .birthDay(form.getBirthDay())
                  .zip(form.getZip())
                  .address(form.getAddress())
                  .build();
          accountService.register(account, form.getPassword()); // (8)
          attributes.addFlashAttribute(account); // (9)
          return "redirect:/account/create?finish"; // (10)
      }

      @RequestMapping(value = "create", params = "finish", method = RequestMethod.GET) // (11)
      String createFinish() {
          return "account/createFinish";
      }
  }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | \ ``AccountService``\ をインジェクションして、主処理を委譲します。
   * - | (2)
     - | 画面で使うフォームに対応したオブジェクトを初期化して、\ ``Model``\ に追加します。\ ``AccountForm``\ は次に説明します。
   * - | (3)
     - | アカウント作成フォーム画面表示処理のためのリクエストマッピングを記述します。
   * - | (4)
     - | アカウント作成処理のためのリクエストマッピングを記述します。
   * - | (5)
     - | 入力されたフォームをバリデーションします。結果は隣の引数の\ ``BindingResult``\ に格納されます。
   * - | (6)
     - | バリデーションエラーがある場合は、フォーム画面に戻ります。
   * - | (7)
     - | フォームオブジェクトからドメインオブジェクトを作成します。
   * - | (8)
     - | \ ``AccountService``\ のアカウント登録処理を実行します。
   * - | (9)
     - | 作成された\ ``Account``\ オブジェクトをリダイレクト先で参照できるようにフラッシュスコープに設定します。
   * - | (10)
     - | アカウント作成完了画面へリダイレクトします。Post-Redirect-Get (PRG)パターンです。
   * - | (11)
     - | アカウント作成完了画面表示処理のためのリクエストマッピングを記述します。

入力フォームの情報は以下の通りです。

.. tabularcolumns:: |p{0.40\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 40 20 40


   * - フィールド名
     - 型
     - 制約条件
   * - | \ ``email``\
     - | \ ``String``\
     - | 必須
       | 1文字以上、100文字以下
       | Email形式
       | 未使用
   * - | \ ``password``\
     - | \ ``String``\
     - | 必須
       | 6文字以上
   * - | \ ``confirmPassword``\
     - | \ ``String``\
     - | 必須
       | \ ``password``\ と同じ値
   * - | \ ``name``\
     - | \ ``String``\
     - | 必須
       | 6文字以上
   * - | \ ``birthDay``\
     - | \ ``LocalDate``\
     - | 必須
   * - | \ ``zip``\
     - | \ ``String``\
     - | 必須
       | 7桁の半角数字
   * - | \ ``address``\
     - | \ ``String``\
     - | 必須
       | 1文字以上、100文字以下


この情報を元に、フォームクラスを作成しましょう。


\ ``src/main/java/jsug/app/account/AccountForm.java``\ を作成して、以下のコードを記述してください。


.. code-block:: java

  package jsug.app.account;

  import jsug.domain.validation.Confirm;
  import jsug.domain.validation.UnusedEmail;
  import lombok.Data;
  import org.hibernate.validator.constraints.Email;
  import org.springframework.format.annotation.DateTimeFormat;

  import javax.validation.constraints.NotNull;
  import javax.validation.constraints.Pattern;
  import javax.validation.constraints.Size;
  import java.io.Serializable;
  import java.time.LocalDate;

  @Data
  @Confirm(field = "password") // (1)
  public class AccountForm implements Serializable {
      @Email // (2)
      @Size(min = 1, max = 100)
      @NotNull
      @UnusedEmail // (3)
      private String email;
      @Size(min = 6)
      @NotNull
      private String password;
      @NotNull
      private String confirmPassword;
      @Size(min = 1, max = 40)
      @NotNull
      private String name;
      @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) // (4)
      @NotNull
      private LocalDate birthDay;
      @NotNull
      @Pattern(regexp = "[0-9]{7}")
      private String zip;
      @Size(min = 1, max = 100)
      @NotNull
      private String address;
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | \ ``@Confirm``\ アノテーションを使って、\ ``password``\ フィールドと\ ``confirmPassword``\ フィールドの値が等しいことを制約にします。
       | このアノテーションはカスタムルールです。バリデータの実装は\ ``jsug.domain.validation.ConfirmValidator``\ クラスを確認してください。
   * - | (2)
     - | \ ``@Email``\ アノテーションを使って、Email形式であることを制約にします。
   * - | (3)
     - | \ ``@UnusedEmail``\ アノテーションを使って、Emailアドレスが使用されいないことを制約にします。
       | このアノテーションはカスタムルールです。バリデータの実装は\ ``jsug.domain.validation.UnusedEmailValidator``\ クラスを確認してください。
   * - | (4)
     - | \ ``@DateTimeFormat``\ アノテーションを使って、入力値の日付文字列を\ ``LocalDate``\ 型に変換する際のフォーマットを指定します。
       | このアノテーションはバリデーション用ではありません。

.. note::

    ここで使用したカスタムルール\ ``@Confirm``\ 、\ ``@UnusedEmail``\ の作成に関しては、
    \ `こちらのドキュメント <http://terasolunaorg.github.io/guideline/5.0.0.RELEASE/ja/ArchitectureInDetail/Validation.html#id13>`_\ を参考にしました。

\ ``App.java``\ を再実行してアプリケーションを起動してください。

http://localhost:8080/account/create?form\ にアクセスすると、アカウント作成フォームが表示されます。
正しく情報を入力すればアカウントが作成でき、その情報を用いてログインできます。
ただし、ログイン後のControllerは未実装なのでエラー画面が表示されます。

GoodsControllerの実装
================================================================================

次は\ ``GoodsController``\ に以下の処理を実装します。

* 商品一覧画面表示処理
* 商品一覧画面表示(カテゴリ指定)処理
* 商品をカートへ追加処理

\ ``src/main/java/jsug/app/goods/GoodsController.java``\ を作成して、以下のコードを記述してください。


.. code-block:: java

  package jsug.app.goods;

  import jsug.domain.model.Cart;
  import jsug.domain.model.Category;
  import jsug.domain.model.Goods;
  import jsug.domain.model.OrderLine;
  import jsug.domain.service.category.CategoryService;
  import jsug.domain.service.goods.GoodsService;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.data.domain.Page;
  import org.springframework.data.domain.Pageable;
  import org.springframework.data.web.PageableDefault;
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.validation.BindingResult;
  import org.springframework.validation.annotation.Validated;
  import org.springframework.web.bind.annotation.*;

  import java.util.List;

  @Controller
  public class GoodsController {

      @Autowired
      GoodsService goodsService;
      @Autowired
      CategoryService categoryService;
      @Autowired
      Cart cart; // (1)

      @ModelAttribute("categories") // (2)
      List<Category> getCategories() {
          return categoryService.findAll();
      }

      @ModelAttribute
      AddToCartForm addToCartForm() {
          return new AddToCartForm();
      }

      @RequestMapping(value = "/")
      String showGoods(@RequestParam(defaultValue = "1") Integer categoryId,
                       @PageableDefault Pageable pageable /* (3) */, Model model) {
          Page<Goods> page = goodsService.findByCategoryId(categoryId, pageable);
          model.addAttribute("page", page);
          model.addAttribute("categoryId", categoryId);
          return "goods/showGoods";
      }

      @RequestMapping(value = "/addToCart", method = RequestMethod.POST)
      String addToCart(@Validated AddToCartForm form, BindingResult result,
                       @PageableDefault Pageable pageable, Model model) {
          if (result.hasErrors()) {
              return showGoods(form.getCategoryId(), pageable, model);
          }
          Goods goods = goodsService.findOne(form.getGoodsId());
          cart.add(OrderLine.builder()
                  .goods(goods)
                  .quantity(form.getQuantity())
                  .build());
          return "redirect:/cart";
      }
  }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | \ ``Cart``\ モデルをインジェクションします。このモデルのBean定義はまだ行っていません。この後、定義します。
   * - | (2)
     - | 画面で常に表示するカテゴリ一覧情報を毎度\ ``Model``\ に設定するために、\ ``@ModelAttribute``\ アノテーションを使っています。
       | この今回は一画面しか使わないので、\ ``model.addAttribute("categories", categoryService.findAll())``\ でも良いです。
   * - | (3)
     - | ページングのための情報を取得します。今回のハンズオンではページングのUIはありませんが、サーバーサイドは念のためページングの用意をしておきます。

商品をカートに追加する際の入力フォームの情報は以下の通りです。

.. tabularcolumns:: |p{0.40\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 40 20 40


   * - フィールド名
     - 型
     - 制約条件
   * - | \ ``goodsId``\
     - | \ ``UUID``\
     - | 必須
   * - | \ ``quantity``\
     - | \ ``Integer``\
     - | 必須
       | 1以上、50以下
   * - | \ ``categoryId``\
     - | \ ``Integer``\
     - | 必須


この情報を元に、フォームクラスを作成しましょう。


\ ``src/main/java/jsug/app/goods/AddToCartForm.java``\ を作成して、以下のコードを記述してください。


.. code-block:: java

  package jsug.app.goods;

  import lombok.Data;

  import javax.validation.constraints.Max;
  import javax.validation.constraints.Min;
  import javax.validation.constraints.NotNull;
  import java.util.UUID;

  @Data
  public class AddToCartForm {
      @NotNull
      private UUID goodsId;
      @NotNull
      @Min(1)
      @Max(50)
      private Integer quantity;
      @NotNull
      private Integer categoryId;
  }

次に、\ ``Cart``\ をDIコンテナに管理させるためにJavaConfigを修正します。

なぜ、DIコンテナに管理させたいかというと、\ ``Cart``\ のスコープを制御したいからです。
ここでは\ ``Cart``\ を、ログインしたユーザーがログイン中に同じカートインスタンスを使い続けられるように、Sessionスコープに登録します。

もし、DIコンテナを使用しなければ、自前で\ ``HttpSession``\ へのオブジェクトの出し入れを管理する必要があり、アプリケーションのソースコードが煩雑になってしまいます。

.. code-block:: java
  :emphasize-lines: 3,10-12,32-36

  package jsug;

  import jsug.domain.model.Cart;
  import net.sf.log4jdbc.sql.jdbcapi.DataSourceSpy;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
  import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.Scope;
  import org.springframework.context.annotation.ScopedProxyMode;
  import org.springframework.web.context.WebApplicationContext;

  import javax.sql.DataSource;

  @Configuration
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
      @Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS) // (1)
      Cart cart() {
          return new Cart();
      }
  }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90


   * - 項番
     - 説明
   * - | (1)
     - | \ ``Cart``\ をBean定義します。\ ``scope``\ 属性には\ ``session``\ を指定します。
       | \ ``session``\ スコープよりも寿命の長い\ ``singleton``\ スコープのControllerにもインジェクションできるように、scoped-proxyの設定を行います。
       | \ ``Cart``\ の振る舞いには特別なインターフェースを用意していないため、\ ``scope``\ 属性には\ ``TARGET_CLASS``\ を指定します。

これでログイン後の画面が表示されるようになりました。カートに商品を追加するとその先のカート一覧画面が404エラーを返すはずです。

CartControllerの実装
================================================================================

次は\ ``CartController``\ に以下の処理を実装します。

* カート表示画面処理
* 商品をカートから削除処理

\ ``src/main/java/jsug/app/cart/CartController.java``\ を作成して、以下のコードを記述してください。


.. code-block:: java

  package jsug.app.cart;

  import jsug.domain.model.Cart;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.validation.BindingResult;
  import org.springframework.validation.annotation.Validated;
  import org.springframework.web.bind.annotation.ModelAttribute;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RequestMethod;

  @Controller
  @RequestMapping("cart")
  public class CartController {
      @Autowired
      Cart cart;

      @ModelAttribute
      CartForm setUpForm() {
          return new CartForm();
      }

      @RequestMapping(method = RequestMethod.GET)
      String viewCart(Model model) {
          model.addAttribute("orderLines", cart.getOrderLines());
          return "cart/viewCart";
      }

      @RequestMapping(method = RequestMethod.POST)
      String removeFromCart(@Validated CartForm cartForm, BindingResult bindingResult, Model model) {
          if (bindingResult.hasErrors()) {
              model.addAttribute("error", "商品がチェックされていません"); // (1)
              return viewCart(model);
          }
          cart.remove(cartForm.getLineNo());
          return "redirect:/cart";
      }
  }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | これまでは入力フォームの横にBean Validationのエラーメッセージを出力していましたが、今回は画面の上部にエラーメッセージを表示します。

カートから商品を削除する際の入力フォームの情報は以下の通りです。

.. tabularcolumns:: |p{0.40\linewidth}|p{0.20\linewidth}|p{0.40\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 40 20 40

   * - フィールド名
     - 型
     - 制約条件
   * - | \ ``lineNo``\ (一覧テーブルの行番号)
     - | \ ``Set<Integer>``\
     - | 1件以上


\ ``src/main/java/jsug/app/cart/CartForm.java``\ を作成して、以下のコードを記述してください。

.. code-block:: java

  package jsug.app.cart;

  import lombok.Data;
  import org.hibernate.validator.constraints.NotEmpty;

  import javax.validation.constraints.NotNull;
  import java.io.Serializable;
  import java.util.Set;

  @Data
  public class CartForm implements Serializable {
      @NotEmpty
      @NotNull
      private Set<Integer> lineNo;
  }

カート確認画面が表示できました。最後に注文画面を実装しましょう。

OrderControllerの実装
================================================================================

次は\ ``OrderController``\ に以下の処理を実装します。

* 注文確認画面処理
* 注文処理
* 注文完了画面表示処理

\ ``src/main/java/jsug/app/order/OrderController.java``\ を作成して、以下のコードを記述してください。

.. code-block:: java

  package jsug.app.order;

  import jsug.domain.model.Cart;
  import jsug.domain.model.Order;
  import jsug.domain.service.order.OrderService;
  import jsug.domain.service.userdetails.ShopUserDetails;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.security.web.bind.annotation.AuthenticationPrincipal;
  import org.springframework.stereotype.Controller;
  import org.springframework.ui.Model;
  import org.springframework.web.bind.annotation.*;
  import org.springframework.web.servlet.mvc.support.RedirectAttributes;

  @Controller
  @RequestMapping("order")
  public class OrderController {
      @Autowired
      OrderService orderService;
      @Autowired
      Cart cart;

      @RequestMapping(method = RequestMethod.GET, params = "confirm")
      String confirm(@AuthenticationPrincipal ShopUserDetails userDetails /* (1) */, Model model) {
          model.addAttribute("orderLines", cart.getOrderLines());
          if (cart.isEmpty()) {
              model.addAttribute("error", "買い物カゴが空です");
              return "cart/viewCart";
          }
          model.addAttribute("account", userDetails.getAccount());
          model.addAttribute("signature", orderService.calcSignature(cart)); // (2)
          return "order/confirm";
      }

      @RequestMapping(method = RequestMethod.POST)
      String order(@AuthenticationPrincipal ShopUserDetails userDetails,
                   @RequestParam String signature, RedirectAttributes attributes) {
          Order order = orderService.purchase(userDetails.getAccount(), cart, signature);
          attributes.addFlashAttribute(order);
          return "redirect:/order?finish";
      }

      @RequestMapping(method = RequestMethod.GET, params = "finish")
      String finish() {
          return "order/finish";
      }
  }



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``@AuthenticationPrincipal``\ アノテーションでログイン中の認証ユーザー情報を取得できます。
       | \ ``account``\ プロパティに、ユーザーの住所などが含まれており、\ ``Model``\ に渡して画面で表示します。
   * - | (2)
     - | 注文確認画面で確認したカートの状態と実際に注文した際の状態に違いがないように署名データを埋め込みます。

これで一通りのアプリケーションの実装が終わりました。

アカウント作成から、商品選択、注文まで動かすことができます。

入力チェックエラーメッセージの変更
================================================================================

いまの実装では、入力チェックがBean Validationのデフォルトのままであまり親切ではありません。

エラーメッセージの変え方は2種類あります。

* Bean Validationの作法(\ ``ValidationMessages.properties``\ )でメッセージを定義する
* Springの作法(\ ``MessageSource``\ )でメッセージを定義する

Springの作法でメッセージを定義する方が、メッセージの粒度(アノテーション単位か、フォーム単位か、フィールド単位か等)を決められたり、
フィールド名を含められたりするので、こちらを使用します。

Spring Bootでは\ ``src/main/resources/messages.properties``\ に定義したメッセージが\ ``MessageSource``\ で使われます。
デフォルトエンコードがUTF-8なので、\ ``native2ascii``\ コマンドで変換する必要はなく、日本語をそのまま記述できます。

\ ``src/main/resources/messages.properties``\ に以下のようなメッセージを定義してください。

.. code-block:: properties

  NotNull=入力必須です
  Size=長さは{2}以上、{1}以下にしてください
  Max={1}以下の値を入力してください
  Min={1}以上の値を入力してください
  Email=E-mailの形式が不正です
  UnusedEmail=既に登録済みです

  Pattern.zip=7桁の整数を入力してください
  Confirm.confirmPassword=パスワードとパスワード(確認)が異なります

メッセージは以下の優先順で解決されます。

1. code + "." + object name + "." + field
2. code + "." + field
3. code + "." + field type
4. code

Bean Validationの場合、codeはアノテーション名になります。

.. note::

    より詳細な情報は以下のページを参照してください。

    * http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.html
    * http://terasolunaorg.github.io/guideline/5.0.0.RELEASE/ja/ArchitectureInDetail/Validation.html#validation-message-def