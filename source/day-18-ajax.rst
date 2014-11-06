18日目: AJAX
============

.. include:: common/original.rst.inc

| 昨日は、 Zend Lucene ライブラリのおかげで、 Jobeet に非常に強力な検索エンジンを実装しました。
| 今日は、検索エンジンの応答性を向上させるために、 AJAX を利用して検索エンジンを動的なものに変換し ます。
| JavaScript が有効でも無効でもフォームが動作するように、ライブ検索機能は控えめな( unobtrusive ) JavaScript を使用して実装されます。
| 控えめな JavaScript を使うことは、クライアントコードの HTML、CSS と JavaScript の振舞いの良好な分離が可能になります。

..
   Yesterday, we implemented a very powerful search engine for Jobeet, thanks to the Zend Lucene library.
   In the following lines, to enhance the responsiveness of the search engine, we will take advantage of AJAX to convert the search engine to a live one.
   As the form should work with and without JavaScript enabled, the live search feature will be implemented using unobtrusive JavaScript.
   Using unobtrusive JavaScript also allows for a better separation of concerns in the client code between HTML, CSS, and the JavaScript behaviors.

jQueryのインストール
--------------------

| jQuery の Web サイトにアクセスし、最新バージョンをダウンロードして、 src/Ibw/JobeetBundle/Resources/public/js/ 以下に .js ファイルを置きます。
| js ディレクトリに .js ファイルを入れた後に、次のコマンドを実行して Symfony に公開状態にするよう指示します。

..
   Go to the jQuery website, download the latest version, and put the .js file under src/Ibw/JobeetBundle/Resources/public/js/.
   After putting the .js file in the js directory, run the following command to tell Symfony to make it available to the public:

.. code-block:: bash

   $ php app/console assets:install web --symlink

jQueryを含める
--------------

すべてのページで jQuery を必要とするため、それが含まれるようにレイアウトを更新します。

.. As we will need jQuery on all pages, update the layout to include it:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       {% block javascripts %}
           <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/jquery-2.0.3.min.js') }}"></script>
       {% endblock %}
   <!-- ... -->

ビヘイビアの追加
----------------

| ライブ検索を実装すると、検索ボックスにユーザーが文字を入力するたびに、サーバーへのコールが行われます。
| サーバは、ページ全体を更新せずに、ページの一部の領域を更新するための必要な情報を返します。
| jQuery の背景にある主要な原則は、 HTML 属性に on*() というビヘイビアを追加するのではなく、
| ページが完全にロードされた後に DOM にビヘイビアを追加することです。
| この方法では、お使いのブラウザが JavaScript のサポートを無効にした場合、
| ビヘイビアは全く登録されませんが、フォームは以前と同じように動作します。
| 最初のステップは、ユーザが検索ボックスにキーを入力することを常時傍受することです。

..
   Implementing a live search means that each time the user types a letter in the search box, a call to the server needs to be triggered;
   the server will then return the needed information to update some regions of the page without refreshing the whole page.
   Instead of adding the behavior with an on*() HTML attributes, the main principle behind jQuery is to add behaviors to the DOM after the page is fully loaded.
   This way, if you disable JavaScript support in your browser, no behavior is registered, and the form still works as before.
   The first step is to intercept whenever a user types a key in the search box:

.. note::

   実装前のコードの説明

   .. Explaining code before implementing

   .. code-block:: javascript

      $('#search_keywords').keyup(function(key)
      {
          if (this.value.length >= 3 || this.value == '')
          {
              // do something
          }
      });

   .. note::

      後で大きく修正するので、今はコードを追加しないでください。
      最終的な JavaScript コードは、次のセクションでレイアウトに追加されます。

      ..
         Don’t add the code for now, as we will modify it heavily.
         The final JavaScript code will be added to the layout in the next section.

| jQuery はユーザーのキー入力のたびに、上記のコードで定義された無名関数を実行します。
| ただし、ユーザーが 3 文字以上入力した場合、または、 input タグからすべてを削除した場合に限ります。
| サーバへの AJAX のコールを作ることは DOM 要素上の load() メソッドを使用するのと同じくらい簡単です。

..
   Every time the user types a key, jQuery executes the anonymous function defined in the above code,
   but only if the user has typed more than 3 characters or if he removed everything from the input tag.
   Making an AJAX call to the server is as simple as using the load() method on the DOM element:

.. note::

   実装前のコードの説明

   .. Explaining code before implementing

   .. code-block:: javascript

      $('#search_keywords').keyup(function(key)
      {
          if (this.value.length >= 3 || this.value == '')
          {
              $('#jobs').load(
                  $(this).parents('form').attr('action'), { query: this.value + '*' }
              );
          }
      });

| AJAX のコールは、通常と同じ呼び出しで行います。
| アクションへの必要な変更は、次のセクションで行います。
| JavaScript が有効になっている場合、少なくとも最後には、検索ボタンを削除したいと思うでしょう。

..
   To manage the AJAX Call, the same action as the “normal” one is called.
   The needed changes in the action will be done in the next section.
   Last but not least, if JavaScript is enabled, we will want to remove the search button:

.. note::

   実装前のコードの説明

   .. Explaining code before implementing

   .. code-block:: javascript

      $('.search input[type="submit"]').hide();

ユーザーのフィードバック
------------------------

| AJAX 呼び出しを行う際、ページはすぐには更新されません。
| ブラウザがページを更新する前に、サーバーの応答を待ちます。
| それまでの間、視覚的なフィードバックを提供し、ユーザーに何かが起きていることを知らせます。
| 慣例では、 AJAX 呼び出しの間に読み込みアイコンを表示します。
| レイアウトに読み込みイメージを追加し、デフォルトではそれを非表示にします。

..
   Whenever you make an AJAX call, the page won’t be updated right away.
   The browser will wait for the server response to come back before updating the page.
   In the meantime, you need to provide visual feedback to the user to inform him that something is going on.
   A convention is to display a loader icon during the AJAX call.
   Update the layout to add the loader image and hide it by default:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       <div class="search">
           <h2>Ask for a job</h2>
           <form action="{{ path('ibw_job_search') }}" method="get">
               <input type="text" name="query" value="{{ app.request.get('query') }}" id="search_keywords" />
               <input type="submit" value="search" />
               <img id="loader" src="{{ asset('bundles/ibwjobeet/images/loader.gif') }}" style="vertical-align: middle; display: none" />
               <div class="help">
                   Enter some keywords (city, country, position, ...)
               </div>
           </form>
       </div>
   <!-- ... -->

| これで HTML を動作させるために必要なすべての部分を持ちましたので、
| search.js ファイルを作成し、これまでに説明した JavaScript を含めます。

.. Now that you have all the pieces needed to make the HTML work, create a search.js file that contains the JavaScript we have explained so far:

src/Ibw/JobeetBundle/Resources/public/js/search.js

.. code-block:: javascript

   $(document).ready(function()
   {
       $('.search input[type="submit"]').hide();

       $('#search_keywords').keyup(function(key)
       {
           if(this.value.length >= 3 || this.value == '') {
               $('#loader').show();
               $('#jobs').load(
                   $(this).parent('form').attr('action'),
                   { query: this.value ? this.value + '*' : this.value },
                   function() {
                       $('#loader').hide();
                   }
               );
           }
       });
   });

Symfony に公開状態にするよう指示するコマンドを実行します。

.. Run the command for telling Symfony to make it available to the public:

.. code-block:: bash

   $ php app/console assets:install web --symlink

また、この新しいファイルが含まれるように、レイアウトを更新する必要があります。

.. You also need to update the layout to include this new file:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       {% block javascripts %}
           <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/jquery-2.0.3.min.js') }}"></script>
           <script type="text/javascript" src="{{ asset('bundles/ibwjobeet/js/search.js') }}"></script>
       {% endblock %}
   <!-- ... -->

アクションの中での AJAX
-----------------------

| JavaScript が有効になっている場合、 jQuery は検索ボックスでのすべてのキー入力を傍受し、 search アクションを呼び出します。
| 有効になっていない場合も、ユーザーがキーを押してフォーム送信し、同じ search アクションを呼び出します。
| そのため、 search アクションが、AJAX 経由での呼び出しか、そうでないかを判断する必要があります。
| リクエストが AJAX でコールされているときは常に、リクエストオブジェクトの isXmlHttpRequest() メソッドが true を返します。

..
   If JavaScript is enabled, jQuery will intercept all keys typed in the search box, and will call the search action.
   If not, the same search action is also called when the user submits the form by pressing the enter key.
   So, the search action now needs to determine if the call is made via AJAX or not.
   Whenever a request is made with an AJAX call, the isXmlHttpRequest() method of the request object returns true.

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   use Symfony\Component\HttpFoundation\Response;

   class JobController extends Controller
   {
       // ...

       public function searchAction(Request $request)
       {
           $em = $this->getDoctrine()->getManager();
           $query = $this->getRequest()->get('query');

           if(!$query) {
               if(!$request->isXmlHttpRequest()) {
                   return $this->redirect($this->generateUrl('ibw_job'));
               } else {
                   return new Response('No results.');
               }
           }

           $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery($query);

           if($request->isXmlHttpRequest()) {

               return $this->render('IbwJobeetBundle:Job:list.html.twig', array('jobs' => $jobs));
           }

           return $this->render('IbwJobeetBundle:Job:search.html.twig', array('jobs' => $jobs));
       }
   }

| 検索が結果を返さない場合は、空白のページの代わりにメッセージを表示する必要があります。
| ここでは、単純な文字列を返します。

..
   If the search returns no result, we need to display a message instead of a blank page.
   We will return just a simple string:

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   public function searchAction(Request $request)
   {
     $em = $this->getDoctrine()->getManager();
     $query = $this->getRequest()->get('query');

     if(!$query) {
         if(!$request->isXmlHttpRequest()) {
             return $this->redirect($this->generateUrl('ibw_job'));
         } else {
             return new Response('No results.');
         }
     }

     $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery($query);

     if($request->isXmlHttpRequest()) {
         if('*' == $query || !$jobs || $query == '') {
             return new Response('No results.');
         }

         return $this->render('IbwJobeetBundle:Job:list.html.twig', array('jobs' => $jobs));
     }

     return $this->render('IbwJobeetBundle:Job:search.html.twig', array('jobs' => $jobs));
   }

AJAX のテスト
-------------

| Symfony のブラウザは JavaScript をシミュレートすることができなため、
| AJAX の呼び出しをテストする際に、手助けしてあげる必要があります。
| これは主に、 jQuery と他のすべての主要な JavaScript ライブラリが送信するリクエストに、
| 手動でヘッダを追加する必要があることを意味します。

..
   As the symfony browser cannot simulate JavaScript, you need to help it when testing AJAX calls.
   It mainly means that you need to manually add the header that jQuery and all other major JavaScript libraries send with the request:

src/Ibw/JobeetBundle/Tests/Controller/JobControllerTst.php

.. code-block:: php

   class JobControllerTest extends WebTestCase
   {
       // ...

       public function testSearch()
       {
           $client = static::createClient();

           $crawler = $client->request('GET', '/job/search');
           $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::searchAction', $client->getRequest()->attributes->get('_controller'));

           $crawler = $client->request('GET', '/job/search?query=sens*', array(), array(), array(
               'X-Requested-With' => 'XMLHttpRequest',
           ));
           $this->assertTrue($crawler->filter('tr')->count()== 2);
       }
   }

| 17 日目では、検索エンジンを実装するために Zend Luceneライブラリを使用しました。
| 今日、それがより敏感にするために jQuery を使用しました。
| Symfony フレームワークは、簡単に MVC アプリケーションを構築するためにすべての基本的なツールを提供し、
| また他のコンポーネントと上手に共存します。
| いつものように、作業のためには最適なツールを使用するようにしてください。
| 明日は、 Jobeet の Web サイトを国際化する方法を説明します。

..
   In day 17, we used the Zend Lucene library to implement the search engine.
   Today, we used jQuery to make it more responsive.
   The symfony framework provides all the fundamental tools to build MVC applications with ease, and also plays well with other components.
   As always, try to use the best tool for the job.
   Tomorrow, we will explain how to internationalize the Jobeet website.

.. include:: common/license.rst.inc
