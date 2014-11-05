5日目: ルーティング
===================

.. include:: common/original.rst.inc

URL
---

| Jobeet ホームページのジョブ情報をクリックすると、 URL は /job/1/show のようになります。
| すでに PHP で Web サイトを開発している場合は、おそらく /job.php?id=1 のような URL に慣れているでしょう。
| Symfony はどのようにそれを動作させるのでしょうか？ この URL に基づいてどのようにアクションを決定するのでしょうか？
| なぜジョブの id は、アクションの $id パラメータを使用して取得されたのでしょうか？
| ここでは、これらすべての質問にお答えします。
| 既に src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig テンプレートに次のコードを見てきました。

.. code-block:: html+jinja

   {{ path('ibw_job_show', { 'id': entity.id }) }}

| これはid 1を持つジョブの URL を生成するために、 ``path`` テンプレートヘルパー関数を使用しています。
| ibw_job_show は使用されるルートの名前です。後述の設定で定義されています。

ルーティング設定
----------------

| Symfony2 では、ルーティング設定は、通常の app/config/routing.yml ファイルで行われます。
| これは、特定のバンドルのルーティング設定をインポートします。
| この例では、 src/Ibw/JobeetBundle/Resources/config/routing.yml ファイルがインポートされます。

app/config/routing.yml

.. code-block:: yaml

   ibw_jobeet:
       resource: "@IbwJobeetBundle/Resources/config/routing.yml"
       prefix:   /

| さて、 JobeetBundle の routing.yml を見れば別のルーティングファイルをインポートしているのが分かるでしょう。
| そのファイルは、ジョブ·コントローラのためのものです。
| また、URLパターン ( /hello/{name} )を持つ ibw_jobeet_homepage という名前のルートも定義しています。

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

| それでは ibw_job_show ルートを詳しく見てみましょう。
| ibw_job_show ルートによって定義されたパターンは、id という名前にワイルドカードの条件が与えられ、 /*/show のような役割を果たします。
| URL /1/show の場合は、コントローラーで利用可能な id の値は、 1 です。
| _controllerパラメータは特殊なキーで、 URLがこのルートと一致した場合に、どのコントローラ/アクションが実行されるべきかを Symfony に伝えます。
| このケースでは、 IbwJobeetBundle の JobController の showAction を実行する必要があります。
| 各コントローラのメソッドの引数として使用可能になる為、ルートパラメーター（たとえば、 {id} ）が特に重要です。

Dev 環境のルーティング設定
--------------------------

| dev の環境は Web デバッグツールバーで使用されるルートが含まれ、 app/config/routing_dev.yml ファイルをロードします。
| （すでに /app/config/routing_dev.php から AcmeDemoBundle のルートに削除しました。1日目の AcmeDemoBundle の削除の仕方 を参照のこと）。
| このファイルは最後にメインの設定ファイル routing.yml を読み込みます。

ルートのカスタマイズ
--------------------

| ブラウザで URL / のリクエストを投げたときに、今は、404 Not Foundエラーが表示されます。
| この URL が、定義されたどのルートとも一致しないためです。
| ibw_jobeet_homepage ルートは URL が  /hello/jobeet と一致したとき、 DefaultController クラスの indexAction() メソッドに送ります。
| これを URL / と一致するように変更して、 JobController クラスの indexAction() メソッドを呼び出すようにしてみましょう。
| 変更を行うには、次のように行います。

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   # ...
   ibw_jobeet_homepage:
       pattern:  /
       defaults: { _controller: IbwJobeetBundle:Job:index }

| キャッシュをクリアし、ブラウザから ``http://jobeet.local`` にアクセスしてください。ジョブのホームページが表示されます。
| これで、レイアウト内の Jobeet のロゴのリンクを、 ibw_jobeet_homepage ルートを使用したものに変更することができます。

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       <h1><a href="{{ path('ibw_jobeet_homepage') }}">
           <img alt="Jobeet Job Board" src="{{ asset('bundles/ibwjobeet/images/logo.jpg') }}" />
       </a></h1>
   <!-- ... -->

もう少し複雑にするため、ジョブページの URL を以下のように、より意味のあるものに変更してみましょう。

    /job/sensio-labs/paris-france/1/web-developer

| Jobeet のことを何も知らなくても、ページを見なくても、 Sensio Labs が Web 開発者をフランスのパリで探していることを、 URL から理解することができます。
| 以下のパターンは、上記の URL と一致します。

    /job/{company}/{location}/{id}/{position}

job.yml ファイルの ibw_job_show ルートを編集します。

src/Ibw/JobeetBundle/Resources/config/routing/job.yml

.. code-block:: yaml

   # ...

   ibw_job_show:
       pattern:  /{company}/{location}/{id}/{position}
       defaults: { _controller: "IbwJobeetBundle:Job:show" }

変更されたルートを動かすためのすべてのパラメータを渡す必要があります。

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   <a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.company, 'location': entity.location, 'position': entity.position }) }}">
       {{ entity.position }}
   </a>
   <!-- ... -->

生成された URL を見てみると、それらをまだ使えるものではありません。

   http://jobeet.local/app_dev.php/job/Sensio Labs/Paris,France/1/Web Developer

| すべての非 ASCII 文字を "-" で置き換えることによって、列の値を "スラグ化"(可読性のある URL に変換) する必要があります。
| Job.php ファイルを開き、クラスに次のメソッドを追加します。

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

| また、ジョブ·クラス定義の前に use 文を追加する必要があります。
| その後、 src/Ibw/JobeetBundle/Utils/Jobeet.php ファイルを作成し、その中に slugify() メソッドを追加します。

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

| 3つの新しい「仮想」アクセサを定義しています： getCompanySlug() 、 getPositionSlug() 、および getLocationSlug() です。
| それらは、対応するカラムの値に slugify() メソッドを適用した後で値を返します。
| ここで、テンプレート内の実際のカラム名をこれらの仮想のものに置き換えます。

src/Ibw/JobeetBundle/views/Job/index.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   <a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug}) }}">
      {{ entity.position }}
   </a>
   <!-- ... -->

ルートの ``requirements``
------------------------

| ルーティングシステムは組み込みの検証機能を持っています。
| ルート定義の ``requirements`` の項目で定義された正規表現によって ``pattern`` の各変数を検証することができます。

src/Ibw/JobeetBundle/Resources/config/routing/job.yml

.. code-block:: yaml

   # ...
   ibw_job_show:
       pattern:  /{company}/{location}/{id}/{position}
       defaults: { _controller: "IbwJobeetBundle:Job:show" }
       requirements:
           id:  \d+

   # ...

上記の ``requirements`` の項目は、 id の値に数値型を強制します。そうでない場合は、ルートが一致しません。

ルートのデバッグ
----------------

| ルートの追加とカスタマイズをする時に、ルートを視覚化し、それに関する詳細な情報を取得できるようにすると便利です。
| アプリケーション内のすべてのルート参照するのに最適な方法は、 ``router:debug`` というコンソールコマンドです。
| プロジェクトのルートから以下のコマンドを実行します。

.. code-block:: bash

   $ php app/console router:debug

| このコマンドは、アプリケーション内に設定されたすべてのルートを一覧で出力します。
| また、コマンドの後にルート名を含めることで、単一ルートの非常に具体的な情報を得ることができます。

.. code-block:: bash

   $ php app/console router:debug ibw_job_show


これで今日のすべてです！ Symfony2 のルーティングシステムの詳細については、``book`` のルーティング章を読んでください。

.. include:: common/license.rst.inc
