5日目: ルーティング
===================

.. include:: common/original.rst.inc

URLs
---------------

/仕事/1/ショー：Jobeetホームページ上の求人情報をクリックすると、URLは次のようになります。すでにPHPでWebサイトを開発している場合は、おそらく/job.php?id=1のようなURLに慣れている。どのようにsymfonyはそれを動作させるのですか？どのようにsymfonyはこのURLを基本とするアクションを決定するのですか？なぜジョブのIDは、アクションの$ idパラメータを使用して取得された？ここでは、これらすべての質問にお答えします。
あなたは既にのsrc / IBW/ JobeetBundle/リソース/ビュー/ジョブ/ index.html.twigテンプレートに次のコードを見てきました。
If you click on a job on the Jobeet homepage, the URL looks like this: /job/1/show. If you have already developed PHP websites, you are probably more accustomed to URLs like /job.php?id=1. How does Symfony make it work? How does Symfony determine the action to call based on this URL? Why is the id of the job retrieved with the $id parameter in the action? Here, we will answer all these questions.
You have already seen the following code in the src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig template:

.. code-block:: html+jinja

   {{ path('ibw_job_show', { 'id': entity.id }) }}

これはibw_job_showあなたは以下に見るような構成に定義されて使用されるルートの名前でID 1を持つジョブのURLを生成するために、パスのテンプレートヘルパー関数を使用しています。
This uses the path template helper function to generate the url for the job which has the id 1. The ibw_job_show is the name of the route used, defined in the configuration as you will see below.

Routing Configuration
---------------------

Symfony2のでは、ルーティング設定は、通常のアプリ/ configに/ routing.ymlファイルで行われます。これは、特定のバンドルルーティング設定をインポートします。この例では、SRC/ IBW/ JobeetBundle/リソース/設定/のrouting.ymlファイルがインポートされます：
In Symfony2, routing configuration is usually done in the app/config/routing.yml. This imports specific bundle routing configuration. In our case, the src/Ibw/JobeetBundle/Resources/config/routing.yml file is imported:

app/config/routing.yml

.. code-block:: yaml

   ibw_jobeet:
       resource: "@IbwJobeetBundle/Resources/config/routing.yml"
       prefix:   /

さて、あなたはそれが別のルーティングファイル、ジョブ·コントローラのための1つをインポートし、ibw_jobeet_homepage/ハロー/{名前} URLパターンのためと呼ばれるルートを定義することをお分かりでしょうのrouting.yml JobeetBundleに見れば：
Now, if you look in the JobeetBundle routing.yml you will see that it imports another routing file, the one for the Job controller and defines a route called ibw_jobeet_homepage for the /hello/{name} URL pattern:

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

それではibw_job_showルートに近い見てみましょう。 ibw_job_showルートによって定義されたパターンは、ワイルドカード名IDが与えられます/ * /ショーのような役割を果たします。
URL /1/ショーのために、ID変数を使用すると、コントローラで使用するために利用可能である1の値を、取得します。
_controllerパラメータが伝える特殊なキーであるsymfonyはどのURLがこのルートと一致した場合に、コントローラ/アクションが実行されるべきである、
私たちのケースでは、IbwJobeetBundle中JobControllerからshowActionを実行する必要があります。
各コントローラのメソッドの引数として使用可能になるので、ルートパラメーター（たとえば、{ID}）が特に重要である。
Let’s have a closer look to the ibw_job_show route. The pattern defined by the ibw_job_show route acts like /*/show where the wildcard is given the name id.
For the URL /1/show, the id variable gets a value of 1, which is available for you to use in your controller.
The _controller parameter is a special key that tells Symfony which controller/action should be executed when a URL matches this route,
in our case it should execute the showAction from the JobController in the IbwJobeetBundle.
The route parameters (e.g. {id}) are especially important because each is made available as an argument to the controller method.

Routing Configuration in Dev Environment
----------------------------------------

devの環境はWebデバッグツールバーで使用されるルートが含まれ、アプリ/ configに/ routing_dev.ymlファイルをロードします（すでに/app/config/routing_dev.phpからAcmeDemoBundleのルートに削除 - を参照して1日目を、どのように削除するにはAcmeDemoBundle）。このファイルをロードし、終了時には、メインのrouting.ymlの設定ファイル。
The dev environment loads the app/config/routing_dev.yml file that contains the routes used by the Web Debug Toolbar (you already deleted the routes for the AcmeDemoBundle from /app/config/routing_dev.php – see Day 1, How to remove the AcmeDemoBundle). This file loads, at the end, the main routing.yml configuration file.

Route Customizations
--------------------

ブラウザで/ URLを要求したときに、今、あなたは404 Not Foundエラーが表示されます。このURLが定義された任意のルートと一致しないためです。私たちは、/ハロー/ JobeetのURLと一致してDefaultController、indexアクションに私たちを送信しibw_jobeet_homepageルートを持っている。それでは/ URLと一致するようにしてJobControllerからindexアクションを呼び出すためにそれを変更してみましょう。変更を行うには、次のようにそれを変更します。
For now, when you request the / URL in a browser, you will get a 404 Not Found error. That’s because this URL does not match any routes defined. We have a  ibw_jobeet_homepage route that matches the /hello/jobeet URL and sends us to the DefaultController, index action. Let’s change it to match the / URL and to call the index action from the JobController. To make the change, modify it to the following:

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   # ...
   ibw_jobeet_homepage:
       pattern:  /
       defaults: { _controller: IbwJobeetBundle:Job:index }

あなたはキャッシュをクリアし、httpさて、もし：ブラウザから//jobeet.local、[ジョブのホームページが表示されます。現在ibw_jobeet_homepageルートを使用するようにレイアウト内のJobeetのロゴのリンクを変更することができます。
Now, if you clear the cache and go to http://jobeet.local from your browser, you will see the Job homepage. We can now change the link of the Jobeet logo in the layout to use the ibw_jobeet_homepage route:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       <h1><a href="{{ path('ibw_jobeet_homepage') }}">
           <img alt="Jobeet Job Board" src="{{ asset('bundles/ibwjobeet/images/logo.jpg') }}" />
       </a></h1>
   <!-- ... -->

何か少し複雑では、のはより意味のあるものに求人ページのURLを変更してみましょう::
For something a bit more involved, let’s change the job page URL to something more meaningful::

    /job/sensio-labs/paris-france/1/web-developer

Jobeetの約、ページを見ずに何も知らなければ、Web開発者は、フランスのパリで動作するためにはSensio Labsが見ているURLから理解することができます。
以下のパターンは、URLと一致::
Without knowing anything about Jobeet, and without looking at the page, you can understand from the URL that Sensio Labs is looking for a Web developer to work in Paris, France.
The following pattern matches such a URL::

    /job/{company}/{location}/{id}/{position}

job.ymlファイルからibw_job_showルートを編集します。
Edit the ibw_job_show route from the job.yml file:

src/Ibw/JobeetBundle/Resources/config/routing/job.yml

.. code-block:: yaml

   # ...

   ibw_job_show:
       pattern:  /{company}/{location}/{id}/{position}
       defaults: { _controller: "IbwJobeetBundle:Job:show" }

今、私たちはそれを動作させるために変更されたルートのすべてのパラメータを渡す必要があります：
Now, we need to pass all the parameters for the changed route for it to work:

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   <a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.company, 'location': entity.location, 'position': entity.position }) }}">
       {{ entity.position }}
   </a>
   <!-- ... -->

あなたが生成されたURLを見てみると、彼らは私たちが彼らになりたいと全くまだありません::
If you have a look at generated URLs, they are not quite yet as we want them to be::

   http://jobeet.local/app_dev.php/job/Sensio Labs/Paris,France/1/Web Developer

私たちは、すべての非ASCII文字を置き換えることによって、列の値を"slugify"する必要があります - 。 Job.phpファイルを開き、クラスに次のメソッドを追加します。
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
その後、SRC/ IBW/ JobeetBundle/ Utilsの/ Jobeet.phpファイルを作成し、その中にslugifyメソッドを追加します。
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

私たちは3つの新しい「仮想」アクセサ定義している：getCompanySlug（）、getPositionSlug（）、およびgetLocationSlugを（）。彼らはそれにslugify（）メソッドを適用した後で対応するカラムの値を返す。さて、あなたは、テンプレート内のこれらの仮想のものによる実際の列名を置き換えることができます。
We have defined three new “virtual” accessors: getCompanySlug(), getPositionSlug(), and getLocationSlug(). They return their corresponding column value after applying it the slugify() method. Now, you can replace the real column names by these virtual ones in the template:

src/Ibw/JobeetBundle/views/Job/index.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   <a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug}) }}">
      {{ entity.position }}
   </a>
   <!-- ... -->

Route Requirements
------------------

ルーティングシステムは組み込みの検証機能を持っています。各パターンの変数はルート定義の要件エントリを使用して定義された正規表現によって検証することができます。
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

上記の要件項目は、数値の値にIDを強制します。そうでない場合は、ルートが一致しません。
The above requirements entry forces the id to be a numeric value. If not, the route won’t match.

Route Debugging
---------------

追加とルートをカスタマイズするが、それは視覚化し、あなたのルートに関する詳細な情報を取得できるようにすると便利です。デバッグコンソールコマンド：アプリケーション内のすべてのルート参照するのに最適な方法は、ルータを介して行われる。プロジェクトのルートから以下を実行して、コマンドを実行します。
While adding and customizing routes, it’s helpful to be able to visualize and get detailed information about your routes. A great way to see every route in your application is via the router:debug console command. Execute the command by running the following from the root of your project:

.. code-block:: bash

   $ php app/console router:debug

コマンドは、アプリケーション内のすべての設定されたルートの有用一覧を出力します。また、コマンドの後にルート名を含めることで、単一の経路上の非常に具体的な情報を得ることができます。
The command will print a helpful list of all the configured routes in your application. You can also get very specific information on a single route by including the route name after the command:

.. code-block:: bash

   $ php app/console router:debug ibw_job_show

最終的な考え
Final Thoughts

つまり、今日のすべてです！ Symfony2のルーティングシステムの詳細については、ルーティング章では、本を読ん形成。
That’s all for today! To learn more about the Symfony2 routing system read the Routing chapter form the book.

.. include:: common/license.rst.inc
