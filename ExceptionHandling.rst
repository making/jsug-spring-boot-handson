例外ハンドリング
********************************************************************************

前章でWebアプリケーションを構築しましたが、特に例外ハンドリングは行っていません。デフォルトの例外ハンドリングが実施され、
白いエラー画面が表示されます。

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

        @ExceptionHandler(GoodsNotFoundException.class)
        @ResponseStatus(HttpStatus.NOT_FOUND)
        String handleGoodsNotFoundException() {
            return "goods/notFound";
        }
    }


OrderControllerの例外ハンドリング
================================================================================

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

        @ExceptionHandler({EmptyCartOrderException.class, InvalidCartOrderException.class})
        @ResponseStatus(HttpStatus.CONFLICT)
        ModelAndView handleOrderException(RuntimeException e) {
            return new ModelAndView("order/error")
                    .addObject("error", e.getMessage());
        }
    }


予期せぬ例外発生時のエラー画面
================================================================================

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

AspectJによる例外ハンドリング
================================================================================

.. code-block:: xml

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>

.. code-block:: java

    package jsug.infra.logging;

    import lombok.extern.slf4j.Slf4j;
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.aspectj.lang.annotation.Around;
    import org.aspectj.lang.annotation.Aspect;
    import org.springframework.stereotype.Component;

    import javax.servlet.http.HttpServletRequest;

    @Component
    @Aspect
    @Slf4j
    public class HandlerExceptionResolverLoggingAspect {
        @Around("execution(* org.springframework.web.servlet.HandlerExceptionResolver.resolveException(..))")
        public Object logException(ProceedingJoinPoint joinPoint) throws Throwable {
            Object ret = joinPoint.proceed();
            HttpServletRequest request = (HttpServletRequest) joinPoint.getArgs()[0];

            if (request.getAttribute("ERROR_LOGGED") == null) {
                Object handler = joinPoint.getArgs()[2];
                Exception exception = (Exception) joinPoint.getArgs()[3];
                log.info("Error occurred [url=" + request.getMethod() + " " + request.getRequestURI() + ", handler=" + handler + "]",
                        exception);
                // mark as logged
                request.setAttribute("ERROR_LOGGED", true);
            }
            return ret;
        }
    }
