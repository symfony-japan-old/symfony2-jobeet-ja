3日目: データモデル
===================

.. include:: common/original.rst.inc

| 今日はいくつかの開発に入ります。テキストエディタを開いて、PHP ファイルを書きたくうずうずしている場合は、これを知ってうれしくなるでしょう。
| 私たちは、 ORM を使用してデータベースと対話し、アプリケーションの最初のモジュールを構築するために、 Jobeet のデータモデルを定義します。
| しかし、 Symfony は、私たちの多くの仕事と同様、 PHP コードをあまり書くことなく、完全に機能するウェブモジュールを持つことになります。

リレーションモデル
------------------

| 前日のユーザーストーリーでは、私たちのプロジェクトのメインオブジェクト（ジョブ、アフィリエイト、およびカテゴリ）について説明しました。
| ここで、対応する ER 図は次のとおりです。

.. image:: /images/Day3-entity_diagram.png

| ストーリーで説明したカラムに加え、 created_at と updated_at のカラムを追加しました。
| オブジェクトが保存または更新されたときに自動的に値を設定するために Symfony を設定します。

データベース
------------

| データベース内の求人、アフィリエイトとカテゴリを格納するために、Symfony 2.3.2 は Doctrine ORM を使用しています。
| データベース接続パラメータを定義するには、（このチュートリアルでは、私たちは、MySQLを使用します） app/config/parameters.yml ファイルを編集する必要があります。

app/config/parameters.yml

.. code-block:: yaml

   parameters:
       database_driver: pdo_mysql
       database_host: localhost
       database_port: null
       database_name: jobeet
       database_user: root
       database_password: password
       # ...

これで Doctrine はあなたのデータベースについて知りました。次のコマンドをターミナルで入力して、データベースを作成することができます。

.. code-block:: bash

    $ php app/console doctrine:database:create

スキーマ
--------

| Doctrine に私たちのオブジェクトを教えるために、「メタデータ」ファイルを作成します。それはオブジェクトをデータベースに格納する方法を説明します。
| 今すぐエディタに移動し、 ``src/Ibw/JobeetBundle/Resources/config`` ディレクトリの中に ``doctrine`` という名前のディレクトリを作成してください。
| ``doctrine`` には 3 つのファイル（Category.orm.yml、Job.orm.ymlとAffiliate.orm.yml）が入っています。

src/Ibw/JobeetBundle/Resources/config/doctrine/Category.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\Category:
       type: entity
       table: category
       id:
           id:
               type: integer
               generator: { strategy: AUTO }
       fields:
           name:
               type: string
               length: 255
               unique: true
       oneToMany:
           jobs:
               targetEntity: Job
               mappedBy: category
       manyToMany:
           affiliates:
               targetEntity: Affiliate
               mappedBy: categories

src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\Job:
       type: entity
       table: job
       id:
           id:
               type: integer
               generator: { strategy: AUTO }
       fields:
           type:
               type: string
               length: 255
               nullable: true
           company:
               type: string
               length: 255
           logo:
               type: string
               length: 255
               nullable: true
           url:
               type: string
               length: 255
               nullable: true
           position:
               type: string
               length: 255
           location:
               type: string
               length: 255
           description:
               type: text
           how_to_apply:
               type: text
           token:
               type: string
               length: 255
               unique: true
           is_public:
               type: boolean
               nullable: true
           is_activated:
               type: boolean
               nullable: true
           email:
               type: string
               length: 255
           expires_at:
               type: datetime
           created_at:
               type: datetime
           updated_at:
               type: datetime
               nullable: true
       manyToOne:
           category:
               targetEntity: Category
               inversedBy: jobs
               joinColumn:
                   name: category_id
                   referencedColumnName: id
       lifecycleCallbacks:
           prePersist: [ setCreatedAtValue ]
           preUpdate: [ setUpdatedAtValue ]

src/Ibw/JobeetBundle/Resources/config/doctrine/Affiliate.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\Affiliate:
       type: entity
       table: affiliate
       id:
           id:
               type: integer
               generator: { strategy: AUTO }
       fields:
           url:
               type: string
               length: 255
           email:
               type: string
               length: 255
               unique: true
           token:
               type: string
               length: 255
           is_active:
               type: boolean
               nullable: true
           created_at:
               type: datetime
       manyToMany:
           categories:
               targetEntity: Category
               joinTable:
                   name: category_affiliate
                   joinColumns:
                       affiliate_id:
                           referencedColumnName: id
                   inverseJoinColumns:
                       category_id:
                           referencedColumnName: id
       lifecycleCallbacks:
           prePersist: [ setCreatedAtValue ]

The ORM
-------

Doctrine は以下のコマンドを使用してオブジェクトを定義するクラスを生成できます。

.. code-block:: bash

    $ php app/console doctrine:generate:entities IbwJobeetBundle

| ``IbwJobeetBundle`` の ``Entity`` ディレクトリを見ると、そこに新しく生成されたクラス（ Category.php 、Job.php と Affiliate.php）があるでしょう。
| Job.php を開いて、``created_at`` と ``updated_at`` に以下のような値を設定します。
I
src/Ibw/JobeetBundle/Entity/Job.php

.. code-block:: php

   // ...

       /**
        * @ORM\PrePersist
        */
       public function setCreatedAtValue()
       {
           if(!$this->getCreatedAt()) {
               $this->created_at = new \DateTime();
           }
       }

       /**
        * @ORM\PreUpdate
        */
       public function setUpdatedAtValue()
       {
           $this->updated_at = new \DateTime();
       }

Affiliate クラスの ``created_at`` の値でも同じようにします。

src/Ibw/JobeetBundle/Entity/Affiliate.php

.. code-block:: php

   // ...

       /**
        * @ORM\PrePersist
        */
       public function setCreatedAtValue()
       {
           $this->created_at = new \DateTime();
       }

   // ...

| これによって、オブジェクトの保存または更新をするときに、 Doctrine が created_at を設定し updated_at を更新するようになります。
| この動作は、上記の Affiliate.orm.yml と Job.orm.yml ファイルで定義されていました。
| また、データベースのテーブルを作成するために以下のコマンドを使用して Doctrine に問い合わせます。

.. code-block:: bash

    $ php app/console doctrine:schema:update --force

.. note::

   このタスクは、開発時に使用されるべきです。体系的に本番データベースを更新する、よりしっかりした方法については、 Doctrine のマイグレーションについて読んでください。

| データベース内にテーブルは作成されていますがデータはそこにありません。
| おおくのWebアプリケーションでは、3つのデータタイプがあります。
| 初期データ（これはアプリケーションが稼動するのに必要。私たちのケースではいくつかの初期カテゴリと管理者ユーザを持つことになります）、
| テストデータ（アプリケーションをテストするために必要）とユーザーデータ（ユーザーがアプリケーションの通常の寿命の間に作成したもの）です。
| いくつかの初期データに基づいてデータベースを作成するためには、 ``DoctrineFixturesBundle``_ を使用します。
| このバンドルのセットアップは、次の手順に従っておこないます。

   1. composer.json ファイルの ``require`` セクションに以下を追加します。

   .. code-block:: json

      // ...
          "require": {
              // ...
              "doctrine/doctrine-fixtures-bundle": "dev-master",
              "doctrine/data-fixtures": "dev-master"
          },

      // ...

   2. ベンダライブラリを更新します。

   .. code-block:: bash

       $ php composer.phar update

   3. app/AppKernel.php にバンドル ``DoctrineFixturesBundle`` を登録します。

      app/AppKernel.php

   .. code-block:: php

      // ...

      public function registerBundles()
      {
          $bundles = array(
              // ...
              new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle()
          );

          // ...
      }

今ではすべてがセットアップされ、私たちはデータをロードするために、私たちのバンドル内の src/Ibw/JobeetBundle/DataFixtures/ORM という名前の新しいフォルダに、いくつかの新しいクラスを作成します。

src/Ibw/JobeetBundle/DataFixtures/ORM/LoadCategoryData.php

.. code-block:: php

   <?php
   namespace Ibw\JobeetBundle\DataFixtures\ORM;

   use Doctrine\Common\Persistence\ObjectManager;
   use Doctrine\Common\DataFixtures\AbstractFixture;
   use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
   use Ibw\JobeetBundle\Entity\Category;

   class LoadCategoryData extends AbstractFixture implements OrderedFixtureInterface
   {
       public function load(ObjectManager $em)
       {
           $design = new Category();
           $design->setName('Design');

           $programming = new Category();
           $programming->setName('Programming');

           $manager = new Category();
           $manager->setName('Manager');

           $administrator = new Category();
           $administrator->setName('Administrator');

           $em->persist($design);
           $em->persist($programming);
           $em->persist($manager);
           $em->persist($administrator);
           $em->flush();

           $this->addReference('category-design', $design);
           $this->addReference('category-programming', $programming);
           $this->addReference('category-manager', $manager);
           $this->addReference('category-administrator', $administrator);
       }

       public function getOrder()
       {
           return 1; // the order in which fixtures will be loaded
       }
   }

src/Ibw/JobeetBundle/DataFixtures/ORM/LoadJobData.php

.. code-block:: php
   <?php
   namespace Ibw\JobeetBundle\DataFixtures\ORM;

   use Doctrine\Common\Persistence\ObjectManager;
   use Doctrine\Common\DataFixtures\AbstractFixture;
   use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
   use Ibw\JobeetBundle\Entity\Job;

   class LoadJobData extends AbstractFixture implements OrderedFixtureInterface
   {
       public function load(ObjectManager $em)
       {
            $job_sensio_labs = new Job();
            $job_sensio_labs->setCategory($em->merge($this->getReference('category-programming')));
            $job_sensio_labs->setType('full-time');
            $job_sensio_labs->setCompany('Sensio Labs');
            $job_sensio_labs->setLogo('sensio-labs.gif');
            $job_sensio_labs->setUrl('http://www.sensiolabs.com/');
            $job_sensio_labs->setPosition('Web Developer');
            $job_sensio_labs->setLocation('Paris, France');
            $job_sensio_labs->setDescription('You\'ve already developed websites with symfony and you want to work with Open-Source technologies. You have a minimum of 3 years experience in web development with PHP or Java and you wish to participate to development of Web 2.0 sites using the best frameworks available.');
            $job_sensio_labs->setHowToApply('Send your resume to fabien.potencier [at] sensio.com');
            $job_sensio_labs->setIsPublic(true);
            $job_sensio_labs->setIsActivated(true);
            $job_sensio_labs->setToken('job_sensio_labs');
            $job_sensio_labs->setEmail('job@example.com');
            $job_sensio_labs->setExpiresAt(new \DateTime('+30 days'));
            $job_extreme_sensio = new Job();
            $job_extreme_sensio->setCategory($em->merge($this->getReference('category-design')));
            $job_extreme_sensio->setType('part-time');
            $job_extreme_sensio->setCompany('Extreme Sensio');
            $job_extreme_sensio->setLogo('extreme-sensio.gif');
            $job_extreme_sensio->setUrl('http://www.extreme-sensio.com/');
            $job_extreme_sensio->setPosition('Web Designer');
            $job_extreme_sensio->setLocation('Paris, France');
            $job_extreme_sensio->setDescription('Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in.');
            $job_extreme_sensio->setHowToApply('Send your resume to fabien.potencier [at] sensio.com');
            $job_extreme_sensio->setIsPublic(true);
            $job_extreme_sensio->setIsActivated(true);
            $job_extreme_sensio->setToken('job_extreme_sensio');
            $job_extreme_sensio->setEmail('job@example.com');
            $job_extreme_sensio->setExpiresAt(new \DateTime('+30 days'));

            $em->persist($job_sensio_labs);
            $em->persist($job_extreme_sensio);
            $em->flush();
       }

       public function getOrder()
       {
           return 2; // the order in which fixtures will be loaded
       }
   }

``fixtures`` が一度書き込まれたら、コマンドラインの ``doctrine:fixtures:load`` コマンドを使用して、それらをロードすることができます。

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

使用しているデータベースをチェックすると、テーブルにロードされたデータが表示されるはずです。

ブラウザーで見る
----------------

| 以下のコマンドを実行し、新しいコントローラー、 src/Ibw/JobeetBundle/Controllers/JobController.php を作成します。
| それは、求人のリスト表示、作成、編集、および削除のアクション（およびそれに対応するテンプレート、フォームとルート）を持ちます。

.. code-block:: bash

    $ php app/console doctrine:generate:crud --entity=IbwJobeetBundle:Job --route-prefix=ibw_job --with-write --format=yml

| このコマンドを実行した後は、いくつかの設定をプロンプトの要求に応じて行う必要があります。デフォルトの答えをただ選択します。
| ブラウザでこれを表示するには、作成した新しいルート src/Ibw/JobeetBundle/Resources/config/routing/job.yml をメインルーティングファイルにインポートする必要があります。

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   IbwJobeetBundle_job:
           resource: "@IbwJobeetBundle/Resources/config/routing/job.yml"
           prefix:   /job
   # ...

また、求人編集フォームからドロップダウンでカテゴリを編集するため、 ``Category`` クラスに ``_toString()`` メソッドを追加する必要があります。

src/Ibw/JobeetBundle/Entity/Category.php

.. code-block:: php

   // ...

   public function __toString()
   {
       return $this->getName() ? $this->getName() : "";
   }

   // ...

キャッシュをクリアします。

.. code-block:: bash

    $ php app/console cache:clear --env=dev
    $ php app/console cache:clear --env=prod

これで、``JobController`` をブラウザで``http://jobeet.local/job/`` （開発環境では ``http://jobeet.local/app_dev.php/job/`` ）にアクセスしてテストすることができます。

.. image:: /images/Day-3-index_page.png

| これで求人の作成、および、編集ができるようになりました。必須フィールドを空白のまま、または、無効なデータを入力しようとしてみてください。
| そう、Symfony はデータベーススキーマの中を調べることで基本的な検証制約を作成しました。
| 以上です。今日、私たちは少ししかPHPコードを書いていませんが、微調整とカスタマイズの準備ができた、 ``job`` モデルのためのWebモジュールを持ちました。
| 明日は、コントローラとビューに親しんでいきます。次回お会いしましょう！

.. include:: common/license.rst.inc

.. _`DoctrineFixturesBundle`: http://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html