4日目: コントローラーとビュー
==================================

.. include:: common/original.rst.inc

今日は、昨日作成した基本的な ``jobController`` をカスタマイズいたします。
そのための必要なコードの大部分はすでに私たちの Jobeet にはあります。

* すべての求人一覧(``list`` )ページ
* 新しい求人作成( ``create`` )ページ
* 既存の求人の更新( ``update`` )をするページ
* 求人の削除( ``delete`` )ページ

そのままのコードで使用はできますが、 `Jobeet mockups`_ に近づくようにリファクタリングします。

MVC アーキテクチャー
--------------------

ウェブ開発のために、今日では、コードを整理するための最も一般的な解決策は、 `MVC design pattern`_ です。
一言で言えば、 MVC デザインパターンは、その性質に応じてコードを整理する方法を定義します。このパターンは、 3 階層にコードを分離します。

* モデル層はビジネスロジックを（データベースはこのレイヤーに所属します）を定義します。
 すでに Symfony には、すべてのクラスとファイルをバンドルの Entity/ ディレクトリ内のモデルに保持しています。

* ビューは、ユーザーと接する部分です。（テンプレートエンジンはこのレイヤーの一部です）
 Symfony 2.3.2 では、Viewレイヤーは主に ``Twig`` テンプレートで構成されています。
 次の文節に出てきまが、それらは、さまざまな Resources/views/ ディレクトリに格納されています。

* コントローラは、クライアントに表示するためのビューに渡すデータを、モデルから取得するコードの一部です。
 このチュートリアルの最初に Symfony をインストールしたときに、すべてのリクエストがフロントコントローラ（app.phpとapp_dev.php）によって管理されていることを見ました。
 これらフロントコントローラはアクションに実際の作業を委任します。


レイアウト
----------

`mockups`_ を詳しく見てみると、各ページのほとんどが同じであることに気づくでしょう。
HTMLやPHPのコードが重複することはよくないことはご存知だと思います。
そのため、共通の表示要素の重複を避けるような方法を見つける必要があります。
この問題を解決する1つの方法は、ヘッダとフッタを定義し、それらを各テンプレートから取り込むことです。
この問題を解決するためのより良い方法は、別の設計パターンである `decorator design pattern`_ を使用することです。
デコレータデザインパターンは、問題を別の方法で、順番を逆にして解決します。
テンプレートは、コンテンツがレイアウトと呼ばれる共通テンプレートによってレンダリングされた後に装飾されます。
Symfony2 には、デフォルトのレイアウトが付随していないため、レイアウトを作成し、アプリケーションのページを修飾するために使用します。
src/Ibw/JobeetBundle/Resources/views/ ディレクトリに新しいファイル layout.html.twig を作成し、次のコードを記入します。：

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html

   <!DOCTYPE html>
   <html>
       <head>
           <title>
               {% block title %}
                   Jobeet - Your best job board
               {% endblock %}
           </title>
           <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
           {% block stylesheets %}
               <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/main.css') }}" type="text/css" media="all" />
           {% endblock %}
           {% block javascripts %}
           {% endblock %}
           <link rel="shortcut icon" href="{{ asset('bundles/ibwjobeet/images/favicon.ico') }}" />
       </head>
       <body>
           <div id="container">
               <div id="header">
                   <div class="content">
                       <h1><a href="{{ path('ibw_job') }}">
                           <img src="{{ asset('bundles/ibwjobeet/images/logo.jpg') }}" alt="Jobeet Job Board" />
                       </a></h1>

                       <div id="sub_header">
                           <div class="post">
                               <h2>Ask for people</h2>
                               <div>
                                   <a href="{{ path('ibw_job') }}">Post a Job</a>
                               </div>
                           </div>

                           <div class="search">
                               <h2>Ask for a job</h2>
                               <form action="" method="get">
                                   <input type="text" name="keywords" id="search_keywords" />
                                   <input type="submit" value="search" />
                                   <div class="help">
                                       Enter some keywords (city, country, position, ...)
                                   </div>
                               </form>
                           </div>
                       </div>
                   </div>
               </div>

              <div id="content">
                  {% for flashMessage in app.session.flashbag.get('notice') %}
                      <div class="flash_notice">
                          {{ flashMessage }}
                      </div>
                  {% endfor %}

                  {% for flashMessage in app.session.flashbag.get('error') %}
                      <div class="flash_error">
                          {{ flashMessage }}
                      </div>
                  {% endfor %}

                  <div class="content">
                      {% block content %}
                      {% endblock %}
                  </div>
              </div>

              <div id="footer">
                  <div class="content">
                      <span class="symfony">
                          <img src="{{ asset('bundles/ibwjobeet/images/jobeet-mini.png') }}" />
                              powered by <a href="http://www.symfony.com/">
                              <img src="{{ asset('bundles/ibwjobeet/images/symfony.gif') }}" alt="symfony framework" />
                          </a>
                      </span>
                      <ul>
                          <li><a href="">About Jobeet</a></li>
                          <li class="feed"><a href="">Full feed</a></li>
                          <li><a href="">Jobeet API</a></li>
                          <li class="last"><a href="">Affiliates</a></li>
                      </ul>
                  </div>
              </div>
          </div>
      </body>
   </html>

Twig ブロック
-------------

Symfony のデフォルトのテンプレートエンジンである ``Twig`` では、ブロックを上記のように定義することができます。
``Twig`` ブロックはデフォルトのコンテンツを持つことが出来ます（例えば、 ``title`` ブロックを見てください）。
それらを子テンプレートで置換または拡張することができます。
作成したレイアウトを利用するために、すべての求人のテンプレート（ src/Ibw/JobeetBundle/Resources/views/Job/ の ``index``, ``edit``, ``new``, ``show``）の
親テンプレート（ ``layout.html.twig`` ）の設定を拡張し、且つ、元の``body`` ブロックの内容で ``content`` ブロックを上書きします。

.. code-block:: jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block content %}
       <!-- original body block code goes here -->
   {% endblock %}

スタイルシート,画像,JavaScript
------------------------------

これはウェブデザインに関するものではないので、すでに Jobeet に使用するすべての必要なアセットを準備しました。
:download:`download the image files </resources/jobeet-images.zip>` アーカイブをダウンロード、解凍し、
src/Ibw/JobeetBundle/Resources/public/images/ ディレクトリに入れてください。
:download:`download the stylesheet </resources/jobeet-css.zip>` アーカイブをダウンロード、解凍し、
src/Ibw/JobeetBundle/Resources/public/css/ ディレクトリに入れてください。
そして、以下のコマンドを実行します。

.. code-block:: bash

    $ php app/console assets:install web --symlink

Symfonyにアセットを公開するよう指示します。
css のフォルダ内を見ると、4つの cssファイル（admin.css、job.css、jobs.cssとのmain.css）を持っていることがわかります。
main.cssは、すべてのJobeetのページで必要なため、 ``layout.html.twig`` の中の stylesheet ブロック内に含めています。
残りは各ページごとに特化した css ファイルで、特定のページのみそれらを必要としています。
テンプレートに新しい css ファイルを追加するには、 stylesheet ブロックを上書きします。
ブロックの中で、新しい css ファイルを追加する前に、親を呼び出します（そのため、main.css と追加の css ファイルを持っているでしょう）。

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/jobs.css') }}" type="text/css" media="all" />
   {% endblock %}

   <!-- rest of the code -->

src/Ibw/JobeetBundle/Resources/views/Job/show.html.twig

.. code-block:: jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/job.css') }}" type="text/css" media="all" />
   {% endblock %}

   <!-- rest of the code -->

求人のホームページのアクション
-------------------------------

各アクションはクラスのメソッドによって表されます。求人のホームページは、クラスが ``JobController`` で、メソッドが ``indexAction()`` となります。
以下では、データベースからすべてのジョブを取得しています。

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

   public function indexAction()
   {
       $em = $this->getDoctrine()->getManager();

       $entities = $em->getRepository('IbwJobeetBundle:Job')->findAll();

       return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
           'entities' => $entities
       ));
   }

   // ...

コードを詳しく見てみましょう。 ``indexAction()`` メソッドは、すべての ``job`` を取得するために、 Doctrine のエンティティマネージャーを取得します。
それらはデータベースからオブジェクトを取得し永続化する処理に責任を持ちます。
そして、エンティティマネージャーからクエリーを作成するレポジトリを取得します。
これは、テンプレート（ビュー）に渡される Job オブジェクトの Doctrine のクラス、 ``ArrayCollection`` を返すします。

求人のホームページのテンプレート
--------------------------------

index.html.twig テンプレートは、すべての求人の HTML テーブルを生成します。ここでは現在のテンプレートのコードは次のとおりです。

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: html+jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/jobs.css') }}" type="text/css" media="all" />
   {% endblock %}

   {% block content %}
       <h1>Job list</h1>

       <table class="records_list">
           <thead>
               <tr>
                   <th>Id</th>
                   <th>Type</th>
                   <th>Company</th>
                   <th>Logo</th>
                   <th>Url</th>
                   <th>Position</th>
                   <th>Location</th>
                   <th>Description</th>
                   <th>How_to_apply</th>
                   <th>Token</th>
                   <th>Is_public</th>
                   <th>Is_activated</th>
                   <th>Email</th>
                   <th>Expires_at</th>
                   <th>Created_at</th>
                   <th>Updated_at</th>
                   <th>Actions</th>
               </tr>
           </thead>
           <tbody>
           {% for entity in entities %}
               <tr>
                   <td><a href="{{ path('ibw_job_show', { 'id': entity.id }) }}">{{ entity.id }}</a></td>
                   <td>{{ entity.type }}</td>
                   <td>{{ entity.company }}</td>
                   <td>{{ entity.logo }}</td>
                   <td>{{ entity.url }}</td>
                   <td>{{ entity.position }}</td>
                   <td>{{ entity.location }}</td>
                   <td>{{ entity.description }}</td>
                   <td>{{ entity.howtoapply }}</td>
                   <td>{{ entity.token }}</td>
                   <td>{{ entity.ispublic }}</td>
                   <td>{{ entity.isactivated }}</td>
                   <td>{{ entity.email }}</td>
                   <td>{% if entity.expiresat %}{{ entity.expiresat|date('Y-m-d H:i:s') }}{% endif%}</td>
                   <td>{% if entity.createdat %}{{ entity.createdat|date('Y-m-d H:i:s') }}{% endif%}</td>
                   <td>{% if entity.updatedat %}{{ entity.updatedat|date('Y-m-d H:i:s') }}{% endif%}</td>
                   <td>
                       <ul>
                           <li>
                               <a href="{{ path('ibw_job_show', { 'id': entity.id }) }}">show</a>
                           </li>
                           <li>
                               <a href="{{ path('ibw_job_edit', { 'id': entity.id }) }}">edit </a>
                           </li>
                       </ul>
                   </td>
               </tr>
           {% endfor %}
           </tbody>
       </table>

       <ul>
           <li>
               <a href="{{ path('ibw_job_new') }}">
                   Create a new entry
               </a>
           </li>
       </ul>
   {% endblock %}

それでは利用可能なのカラムのセットのみを表示するように整理してみましょう。
以下の ``twig`` のブロックの内容を置き換えます。：

.. code-block:: html+jinja

   {% block content %}
       <div id="jobs">
           <table class="jobs">
               {% for entity in entities %}
                   <tr class="{{ cycle(['even', 'odd'], loop.index) }}">
                       <td class="location">{{ entity.location }}</td>
                       <td class="position">
                           <a href="{{ path('ibw_job_show', { 'id': entity.id }) }}">
                               {{ entity.position }}
                           </a>
                       </td>
                       <td class="company">{{ entity.company }}</td>
                   </tr>
               {% endfor %}
           </table>
       </div>
   {% endblock %}

.. image:: /images/Day-4-2-jobs.png

求人ページテンプレート
----------------------

今度は、求人ページのテンプレートをカスタマイズしましょう。show.html.twig ファイルを開き、次のコードを使用して、その内容を置き換えます。

src/Ibw/JobeetBundle/Resources/views/Job/show.html.twig

.. code-block:: html+jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block title %}
       {{ entity.company }} is looking for a {{ entity.position }}
   {% endblock %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/job.css') }}" type="text/css" media="all" />
   {% endblock %}

   {% block content %}
       <div id="job">
           <h1>{{ entity.company }}</h1>
           <h2>{{ entity.location }}</h2>
           <h3>
               {{ entity.position }}
               <small> - {{ entity.type }}</small>
           </h3>

           {% if entity.logo %}
               <div class="logo">
                   <a href="{{ entity.url }}">
                       <img src="/uploads/jobs/{{ entity.logo }}"
                           alt="{{ entity.company }} logo" />
                   </a>
               </div>
           {% endif %}

           <div class="description">
               {{ entity.description|nl2br }}
           </div>

           <h4>How to apply?</h4>

           <p class="how_to_apply">{{ entity.howtoapply }}</p>

           <div class="meta">
               <small>posted on {{ entity.createdat|date('m/d/Y') }}</small>
           </div>

           <div style="padding: 20px 0">
               <a href="{{ path('ibw_job_edit', { 'id': entity.id }) }}">
                   Edit
               </a>
           </div>
       </div>
   {% endblock %}

.. image:: /images/Day-4-individual-job.png

求人詳細ページアクション
------------------------

求人ページは show アクションによって生成されます。 ``JobController`` の ``showAction()`` メソッドで定義されます。

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   public function showAction($id)
   {
       $em = $this->getDoctrine()->getManager();

       $entity = $em->getRepository('IbwJobeetBundle:Job')->find($id);

       if (!$entity) {
           throw $this->createNotFoundException('Unable to find Job entity.');
       }

       $deleteForm = $this->createDeleteForm($id);

       return $this->render('IbwJobeetBundle:Job:show.html.twig', array(
           'entity' => $entity,
           'delete_form' => $deleteForm->createView(),
       ));
   }

indexアクションと同様に、``IbwJobeetBundle`` のリポジトリクラスは、``job`` を取得するために使用されます。ここでは、``find()`` メソッドを使用しています。
このメソッドのパラメータは、``job`` の一意の識別子である主キーです。
``actionShow()``  関数の ``$id`` パラメータに ``job`` の主キーが含まれている理由を次のセクションで説明します。
データベースにジョブが存在しない場合、$this->createNotFoundException() の例外を投げることで、ユーザーを 404 ページに転送します。
例外として、ユーザーに表示されるページは prod 環境と dev 環境で異なります。

.. image:: /images/Day-4-error1.png
.. image:: /images/Day-4-error2.png

今日はこれですべてです！明日は、ルーティング機能に詳しくなりましょう。

.. include:: common/license.rst.inc

.. _`Jobeet mockups`: http://symfony.com/legacy/doc/jobeet/1_4/en/02?orm=Doctrine#chapter_02_the_project_user_stories
.. _`MVC design pattern`: http://en.wikipedia.org/wiki/Model-view-controller
.. _`mockups`: http://symfony.com/legacy/doc/jobeet/1_4/en/02?orm=Doctrine#chapter_02_the_project_user_stories
.. _`decorator design pattern`: http://en.wikipedia.org/wiki/Decorator_pattern
