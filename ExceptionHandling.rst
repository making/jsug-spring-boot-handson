例外ハンドリングの実装
********************************************************************************

前章でWebアプリケーションを構築しましたが、特に例外ハンドリングは行っていません。デフォルトの例外ハンドリングが実施され、
デフォルトのエラー画面として白いエラー画面が表示されます。

サービスクラスでは以下の条件で例外がスローされます。

.. list-table::
   :header-rows: 1
   :widths: 30 30 40

   * - サービスクラス
     - 例外クラス
     - 発生条件
   * - \ ``GoodsServce``\
     - \ ``GoodsNotFoundException``\
     - 対象の商品が存在しない場合に発生
   * - \ ``OrderService``\
     - \ ``EmptyCartOrderException``\
     - 注文時にカートが空の場合に発生
   * - \ ``OrderService``\
     - \ ``InvalidCartOrderException``\
     - 注文時に注文確認時とカートの内容が異なっている場合に発生

これらの場面で適切な画面に遷移するように、例外ハンドリングを実装しましょう。

本章完了時点でのソースコードツリーを以下に示します。ハイライトされた部分が本章で追加・修正するファイルです。

.. code-block:: console
    :emphasize-lines: 19,23,63-65,114

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



GoodsControllerの例外ハンドリング
================================================================================


Controllerで発生する例外を、Controller単位でハンドリングする場合は\ ``@ExceptionHandler``\ アノテーションを使います。

まずは\ ``GoodsController``\ で発生する\ ``GoodsNotFoundException``\ をハンドリングしましょう。

.. code-block:: java
    :emphasize-lines: 8,14,65-70

    package jsug.app.goods;

    import jsug.domain.model.Cart;
    import jsug.domain.model.Category;
    import jsug.domain.model.Goods;
    import jsug.domain.model.OrderLine;
    import jsug.domain.service.category.CategoryService;
    import jsug.domain.service.goods.GoodsNotFoundException;
    import jsug.domain.service.goods.GoodsService;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.data.web.PageableDefault;
    import org.springframework.http.HttpStatus;
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
        Cart cart;

        @ModelAttribute("categories")
        List<Category> getCategories() {
            return categoryService.findAll();
        }

        @ModelAttribute
        AddToCartForm addToCartForm() {
            return new AddToCartForm();
        }

        @RequestMapping(value = "/")
        String showGoods(@RequestParam(defaultValue = "1") Integer categoryId,
                         @PageableDefault Pageable pageable, Model model) {
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

        @ExceptionHandler(GoodsNotFoundException.class) // (1)
        @ResponseStatus(HttpStatus.NOT_FOUND) // (2)
        String handleGoodsNotFoundException() {
            return "goods/notFound"; // (3)
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``@ExceptionHandler``\ アノテーションでハンドリングする例外クラスを指定します。
       | ここでは\ ``GoodsNotFoundException``\ を対処します。
   * - | (2)
     - | \ ``@ResponseStatus``\ アノテーションでHTTPステータスコードを指定します。ここでは404エラーになるようにします。
       | このアノテーションは例外ハンドリング専用ではなく、 \ ``@RequestMapping``\ アノテーションと組み合わせることもできます。
   * - | (3)
     - | エラー画面のview名(htmlパス)を返します。

このエラーを発生させるのは少し難しいですが、カートに商品を追加する際のPOSTのリクエストを改ざんして、\ ``goodsId``\ の値を変更すれば良いです。

OrderControllerの例外ハンドリング
================================================================================

次に\ ``OrderController``\ で発生する\ ``EmptyCartOrderException``\ と\ ``InvalidCartOrderException``\ をハンドリングします。

.. code-block:: java
    :emphasize-lines: 5-6,10,15,50-56

    package jsug.app.order;

    import jsug.domain.model.Cart;
    import jsug.domain.model.Order;
    import jsug.domain.service.order.EmptyCartOrderException;
    import jsug.domain.service.order.InvalidCartOrderException;
    import jsug.domain.service.order.OrderService;
    import jsug.domain.service.userdetails.ShopUserDetails;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.http.HttpStatus;
    import org.springframework.security.web.bind.annotation.AuthenticationPrincipal;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.*;
    import org.springframework.web.servlet.ModelAndView;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;

    @Controller
    @RequestMapping("order")
    public class OrderController {
        @Autowired
        OrderService orderService;
        @Autowired
        Cart cart;

        @RequestMapping(method = RequestMethod.GET, params = "confirm")
        String confirm(@AuthenticationPrincipal ShopUserDetails userDetails, Model model) {
            model.addAttribute("orderLines", cart.getOrderLines());
            if (cart.isEmpty()) {
                model.addAttribute("error", "買い物カゴが空です");
                return "cart/viewCart";
            }
            model.addAttribute("account", userDetails.getAccount());
            model.addAttribute("signature", orderService.calcSignature(cart));
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

        @ExceptionHandler({EmptyCartOrderException.class, InvalidCartOrderException.class}) // (1)
        @ResponseStatus(HttpStatus.CONFLICT) // (2)
        ModelAndView handleOrderException(RuntimeException e /* (3) */) {
            return new ModelAndView("order/error") // (4)
                    .addObject("error", e.getMessage());
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``@ExceptionHandler``\ アノテーションでハンドリングする例外クラスを複数指定できます。
   * - | (2)
     - | \ ``@ResponseStatus``\ アノテーションで409エラーになるようにします。
   * - | (3)
     - | \ ``@ExceptionHandler``\ で指定した複数のクラスをどちらでも引数でとれるように、共通の親クラスである \ ``RuntimeException``\ で受けます。
   * - | (4)
     - | \ ``@ExceptionHandler``\ メソッドでは\ ``Model``\ が取れない(構築されていない)ので、\ ``ModelAndView``\ を返して\ ``Model``\ とview名を設定します。
       | (今後、\ ``Model``\ を引数にとれるようになる可能性があります)

.. note::

    \ ``@ExceptionHandler``\ は指定したクラスの子クラスもハンドリングできるので、この2つの共通親クラスである、
    \ ``RuntimeException``\ をハンドリングすることで、両クラスをハンドリングできます。この場合は、当然ですが\ ``DataAccessException``\
    などその他の例外もハンドリングされることに注意してください。

注文確認画面まで進んだあと、別のタブを開いてカートの状態を変更して元の画面から注文を行ってみてください。

予期せぬ例外発生時のエラー画面
================================================================================

前述の例外ハンドリングで扱った例外クラス以外の例外が発生した場合は、依然として白いエラー画面が表示されます。
この画面を変更する場合は、view名が\ ``error``\ のViewを用意すれば良いです。

Thymeleafの場合、\ ``src/main/resources/templates/error.html``\ を作成すれば白いエラー画面を上書きできます。

以下のHTMLを作成しましょう。

.. code-block:: html

    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8"/>
        <title>エラー</title>
        <link rel="stylesheet" type="text/css" th:href="@{/css/wro.css}" href="../static/css/wro.css"/>
    </head>
    <body>

    <nav class="navbar navbar-default" role="navigation">
        <div class="container">
            <div class="navbar-header">
                <a class="navbar-brand" th:href="@{/}"><span></span></a>
            </div>
            <div class="collapse navbar-collapse">
                <ul class="nav navbar-nav navbar-right">
                    <li>
                        <a th:href="@{/}" href="goods/showGoods.html">ホーム</a>
                    </li>
                    <li>
                        <a th:href="@{/cart}" href="cart/viewCart.html">カート</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    <br/>

    <div class="container">
        <div class="alert alert-danger">
            エラーが発生しました。<a th:href="@{/}" href="goods/showGoods.html">ホーム</a>に戻ってやり直してください。
        </div>
    </div>
    </body>
    </html>

たとえば、http://localhost:8080/?categoryId=a\ にアクセスすれば、予期せぬエラーが発生しこの画面に遷移します。

AspectJによる例外ハンドリング
================================================================================

先ほどのデフォルトエラー画面を変更しましたが、この影響でデフォルトエラー画面に遷移する際にログが出なくなってしまいました。

そこで、AOPを使ってエラーを捕捉し、ログを出力します。今回はAspectJを使ってAspectを実装します。

Spring MVCの例外ハンドリングは複数の\ ``org.springframework.web.servlet.HandlerExceptionResolver``\ インターフェースによって行われます。
\ ``HandlerExceptionResolver``\ は複数登録でき、連鎖的に呼ばれます。

そこで、このインターフェースに対して、Aspectを作成しログ出力を差し込みます。

pom.xmlに以下の依存関係を追加してください。

.. code-block:: xml

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>

\ ``src/main/java/jsug/infra/logging/HandlerExceptionResolverLoggingAspect``\ を作成し、以下のコードを実装してください。

.. code-block:: java

    package jsug.infra.logging;

    import lombok.extern.slf4j.Slf4j;
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.aspectj.lang.annotation.Around;
    import org.aspectj.lang.annotation.Aspect;
    import org.springframework.stereotype.Component;

    import javax.servlet.http.HttpServletRequest;

    @Component
    @Aspect // (1)
    @Slf4j
    public class HandlerExceptionResolverLoggingAspect {
        @Around("execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))") // (2)
        public Object logException(ProceedingJoinPoint joinPoint /* (3) */) throws Throwable {
            Object ret = joinPoint.proceed();
            HttpServletRequest request = (HttpServletRequest) joinPoint.getArgs()[0]; // (1)

            if (request.getAttribute("ERROR_LOGGED") == null) {
                Object handler = joinPoint.getArgs()[2];
                Exception exception = (Exception) joinPoint.getArgs()[3];
                log.info("Error occurred [url=" + request.getMethod() + " " + request.getRequestURI() + ", handler=" + handler + "]",
                        exception);
                // mark as logged
                request.setAttribute("ERROR_LOGGED", true); // (4)
            }
            return ret;
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``@Aspect``\ アノテーションをつけてAspectを定義します。
   * - | (2)
     - | \ ``@Around``\ アノテーションをつけてAroundアドバイスを定義します。\ ``HandlerExceptionResolver#resolveException``\ メソッドが対象になるように設定します。
   * - | (3)
     - | JointPoint情報から、実行されたメソッドの情報を取得できます。ここでは引数を取得しています。
   * - | (4)
     - | \ ``HandlerExceptionResolver#resolveException``\ メソッドは複数回呼ばれる可能性があるため、マークをつけて複数回呼ばれないようにします。

.. note::

    この例外ハンドリング方法が適切かどうか怪しいですが、汎用的なハンドリング方法として覚えておいて損はないです。

    AOPの用語に関しては\ `こちら <http://d.hatena.ne.jp/minokuba/20110301/1298996870>`_\ を確認してください。


サイド、http://localhost:8080/?categoryId=a\ にアクセスして、ログが出力されることを確認してください。