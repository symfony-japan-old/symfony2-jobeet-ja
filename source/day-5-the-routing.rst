5日目: ルーティング
===================

.. include:: common/original.rst.inc

URL
---

Jobeetトップページの求人情報をクリックすると、URLは /job/1/show のようになります。
すでに PHP で Webサイトを開発している場合は、おそらく /job.php?id=1 のような URL に慣れているでしょう。
Symfony はどのようにそれを動作させるのでしょうか？ このURLに基づいてどのようにアクションを決定するのでしょうか？
なぜジョブの id は、アクションの $id パラメータを使用して取得されたのでしょうか？
ここでは、これらすべての質問にお答えします。
あなたは既に src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig テンプレートに次のコードを見てきました。

.. code-block:: html+jinja

   {{ path('ibw_job_show', { 'id': entity.id }) }}

これはid 1を持つジョブのURLを生成するために、パステンプレートヘルパー関数を使用しています。
ibw_job_show は使用されるルートの名前です。後述の設定で定義されています。

ルーティング設定
----------------

Symfony2 では、ルーティング設定は、通常の app/config/routing.yml ファイルで行われます。
これは、特定のバンドルのルーティング設定をインポートします。
この例では、 src/Ibw/JobeetBundle/Resources/config/routing.yml ファイルがインポートされます。

app/config/routing.yml

.. code-block:: yaml

   ibw_jobeet:
       resource: "@IbwJobeetBundle/Resources/config/routing.yml"
       prefix:   /

さて、JobeetBundleのrouting.yml を見れば別のルーティングファイルをインポートしているのが分かるでしょう。
そのファイルは、ジョブ·コントローラのためのものです。また、URLパターン (/hello/{name} )を持つ ibw_jobeet_homepage という名前のルートも定義しています。

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   IbwJobeetBundle_job:
       resource: "@IbwJobeetBundle/Resources/config/routing/job.yml"
       prefix: /job

   ibw_jobeet_homepage:
       pattern:  /hello/{name}
       defaults: { _controller: IbwJobeetBundle:Default:index }

src/Ibw/JobeetBundle/Resources/config/routing/job.yml

.. code-block:: yaml

   ibw_job:
       pattern:  /
       defaults: { _controller: "IbwJobeetBundle:Job:index" }

   ibw_job_show:
       pattern:  /{id}/show
       defaults: { _controller: "IbwJobeetBundle:Job:show" }

   ibw_job_new:
       pattern:  /new
       defaults: { _controller: "IbwJobeetBundle:Job:new" }

   ibw_job_create:
       pattern:  /create
       defaults: { _controller: "IbwJobeetBundle:Job:create" }
       requirements: { _method: post }

   ibw_job_edit:
       pattern:  /{id}/edit
       defaults: { _controller: "IbwJobeetBundle:Job:edit" }

   ibw_job_update:
       pattern:  /{id}/update
       defaults: { _controller: "IbwJobeetBundle:Job:update" }
       requirements: { _method: post|put }

   ibw_job_delete:
       pattern:  /{id}/delete
       defaults: { _controller: "IbwJobeetBundle:Job:delete" }
       requirements: { _method: post|delete }

それでは ibw_job_show ルートを詳しく見てみましょう。
ibw_job_show ルートによって定義されたパターンは、id という名前にワイルドカードの条件が与えられ、 /*/show のような役割を果たします。
URL /1/show の場合は、コントローラーで利用可能なid の値は、 1 です。
_controllerパラメータは特殊なキーで、 URLがこのルートと一致した場合に、どのコントローラ/アクションが実行されるべきかを Symfony に伝えます。
私たちのケースでは、IbwJobeetBundle の JobController の showAction を実行する必要があります。
各コントローラのメソッドの引数として使用可能になる為、ルートパラメーター（たとえば、 {id} ）が特に重要です。

Dev 環境のルーティング設定
--------------------------

devの環境はWebデバッグツールバーで使用されるルートが含まれ、 app/config/routing_dev.yml ファイルをロードします。
（すでに/app/config/routing_dev.php からAcmeDemoBundleのルートに削除しました。1日目の AcmeDemoBundle の削除の仕方 を参照のこと）。
このファイルは最後にメインの設定ファイル routing.yml を読み込みます。
The dev environment loads the app/config/routing_dev.yml file that contains the routes used by the Web Debug Toolbar
 (you already deleted the routes for the AcmeDemoBundle from /app/config/routing_dev.php – see Day 1, How to remove the AcmeDemoBundle).
 This file loads, at the end, the main routing.yml configuration file.

ルートのカスタマイズ
--------------------

ブラウザで URL / のリクエストを投げたときに、今は、404 Not Foundエラーが表示されます。
このURLが、定義されたどのルートとも一致しないためです。
ibw_jobeet_homepage ルートは URL が  /hello/jobeet と一致したとき、 DefaultController 、 index アクションに送ります。
これを URL / と一致するように変更して、 JobController の index アクションを呼び出すようにしてみましょう。
変更を行うには、次のように行います。

For now, when you request the / URL in a browser, you will get a 404 Not Found error.
That’s because this URL does not match any routes defined.
We have a  ibw_jobeet_homepage route that matches the /hello/jobeet URL and sends us to the DefaultController, index action.
Let’s change it to match the / URL and to call the index action from the JobController.
 To make the change, modify it to the following:

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   # ...
   ibw_jobeet_homepage:
       pattern:  /
       defaults: { _controller: IbwJobeetBundle:Job:index }

キャッシュをクリアし、ブラウザから ``http://jobeet.local`` にアクセスしてください。求人のトップページが表示されます。
レイアウト内の Jobeet のロゴのリンクを、 ibw_jobeet_homepage ルートを使用するように変更することができます。
Now, if you clear the cache and go to ``http://jobeet.local`` from your browser, you will see the Job homepage.
We can now change the link of the Jobeet logo in the layout to use the ibw_jobeet_homepage route:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       <h1><a href="{{ path('ibw_jobeet_homepage') }}">
           <img alt="Jobeet Job Board" src="{{ asset('bundles/ibwjobeet/images/logo.jpg') }}" />
       </a></h1>
   <!-- ... -->

もう少し複雑にするため、求人ページのURLをより意味のあるものに変更してみましょう。
For something a bit more involved, let’s change the job page URL to something more meaningful::

    /job/sensio-labs/paris-france/1/web-developer

Jobeetのことを何も知らなくても、ページを見なくても、Sensio LabsがWeb開発者をフランスのパリで探していることを、URLから理解することができます。
以下のパターンは、上記のURLと一致します。
Without knowing anything about Jobeet, and without looking at the page, you can understand from the URL that Sensio Labs is looking for a Web developer to work in Paris, France.
The following pattern matches such a URL::

    /job/{company}/{location}/{id}/{position}

job.yml ファイルの ibw_job_show ルートを編集します。
Edit the ibw_job_show route from the job.yml file:

src/Ibw/JobeetBundle/Resources/config/routing/job.yml

.. code-block:: yaml

   # ...

   ibw_job_show:
       pattern:  /{company}/{location}/{id}/{position}
       defaults: { _controller: "IbwJobeetBundle:Job:show" }

変更されたルートを動かすためのすべてのパラメータを渡す必要があります。
Now, we need to pass all the parameters for the changed route for it to work:

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   <a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.company, 'location': entity.location, 'position': entity.position }) }}">
       {{ entity.position }}
   </a>
   <!-- ... -->

生成されたURLを見てみると、それらをまだ使えるものではありません。
If you have a look at generated URLs, they are not quite yet as we want them to be::

   http://jobeet.local/app_dev.php/job/Sensio Labs/Paris,France/1/Web Developer

私たちは、すべての非ASCII文字を "-" に置き換えることによって、列の値を "スラグ化"(可読性のあるURLに変換) する必要があります。
Job.phpファイルを開き、クラスに次のメソッドを追加します。
We need to “slugify” the column values by replacing all non ASCII characters by a -. Open the Job.php file and add the following methods to the class:

src/Ibw/JobeetBundle/Entity/Job.php

.. code-block:: php

   // ...
   use Ibw\JobeetBundle\Utils\Jobeet as Jobeet;

   class Job
   {
       // ...

       public function getCompanySlug()
       {
           return Jobeet::slugify($this->getCompany());
       }

       public function getPositionSlug()
       {
           return Jobeet::slugify($this->getPosition());
       }

       public function getLocationSlug()
       {
           return Jobeet::slugify($this->getLocation());
       }
   }

また、ジョブ·クラス定義の前にuse文を追加する必要があります。

その後、src/Ibw/JobeetBundle/Utils/Jobeet.php ファイルを作成し、その中に slugify メソッドを追加します。
You must also add the use statement before the Job class definition.
After that, create the src/Ibw/JobeetBundle/Utils/Jobeet.php file and add the slugify method in it:

src/Ibw/JobeetBundle/Utils/Jobeet.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Utils;

   class Jobeet
   {
       static public function slugify($text)
       {
           // replace all non letters or digits by -
           $text = preg_replace('/\W+/', '-', $text);

           // trim and lowercase
           $text = strtolower(trim($text, '-'));

           return $text;
       }
   }

私たちは3つの新しい「仮想」アクセサを定義しています：getCompanySlug()、getPositionSlug()、および getLocationSlug() です。
彼らはそれに slugify() メソッドを適用した後で対応するカラムの値を返します。
ここで、テンプレート内の実際のカラム名をこれらの仮想のものに置き換えます。
We have defined three new “virtual” accessors: getCompanySlug(), getPositionSlug(), and getLocationSlug().
They return their corresponding column value after applying it the slugify() method.
Now, you can replace the real column names by these virtual ones in the template:

src/Ibw/JobeetBundle/views/Job/index.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   <a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug}) }}">
      {{ entity.position }}
   </a>
   <!-- ... -->

ルートの ``requirements``
------------------------

ルーティングシステムは組み込みの検証機能を持っています。ルート定義の ``requirements`` の項目で定義された正規表現によって``pattern`` の各変数を検証することができます。
The routing system has a built-in validation feature. Each pattern variable can be validated by a regular expression defined using the requirements entry of a route definition:

src/Ibw/JobeetBundle/Resources/config/routing/job.yml

.. code-block:: yaml

   # ...
   ibw_job_show:
       pattern:  /{company}/{location}/{id}/{position}
       defaults: { _controller: "IbwJobeetBundle:Job:show" }
       requirements:
           id:  \d+

   # ...

上記の ``requirements`` の項目は、id の値に数値型を強制します。そうでない場合は、ルートが一致しません。
The above requirements entry forces the id to be a numeric value. If not, the route won’t match.

ルートのデバッグ
----------------

ルートの追加とカスタマイズをする時に、ルートを視覚化し、それに関する詳細な情報を取得できるようにすると便利です。
アプリケーション内のすべてのルート参照するのに最適な方法は、 ``router:debug`` というコンソールコマンドです。
プロジェクトのルートから以下のコマンドを実行します。
While adding and customizing routes, it’s helpful to be able to visualize and get detailed information about your routes.
A great way to see every route in your application is via the router:debug console command.
Execute the command by running the following from the root of your project:

.. code-block:: bash

   $ php app/console router:debug

コマンドは、アプリケーション内のすべての設定されたルート一覧を出力します。
また、コマンドの後にルート名を含めることで、単一ルートの非常に具体的な情報を得ることができます。
The command will print a helpful list of all the configured routes in your application.
You can also get very specific information on a single route by including the route name after the command:

.. code-block:: bash

   $ php app/console router:debug ibw_job_show

最終的な考え
Final Thoughts

これで今日のすべてです！ Symfony2のルーティングシステムの詳細については、``book`` のルーティング章を読んでください。
That’s all for today! To learn more about the Symfony2 routing system read the Routing chapter form the book.

.. include:: common/license.rst.inc
