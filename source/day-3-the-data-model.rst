3日目: データモデル
===================
Day 3: The Data Model
=====================

.. include:: common/original.rst.inc

今日はいくつかの開発に入ります。テキストエディタを開いて、PHP ファイルを書きたくうずうずしている場合は、これを知ってうれしくなるでしょう。
私たちは、 ORM を使用してデータベースと対話し、アプリケーションの最初のモジュールを構築するために、Jobeetのデータモデルを定義します。
しかし、Symfony は、私たちの多くの仕事と同様、PHPコードをあまり書くことなく、完全に機能するWebモジュールを持つことになります。
If you’re itching to open your text editor and lay down some PHP, you will be happy to know that today will get us into some development.
We will define the Jobeet data model, use an ORM to interact with the database and build the first module of the application.
But as Symfony does a lot of work for us, we will have a fully functional web module without writing too much PHP code.

リレーションモデル
------------------
The Relational Model
---------------

前日からのユーザーストーリーは、私たちのプロジェクトのメインオブジェクト（ジョブ、アフィリエイト、およびカテゴリ）について説明しました。
ここで、対応する ER 図は次のとおりです。
The user stories from the previous day describe the main objects of our project: jobs, affiliates, and categories.
Here is the corresponding entity relationship diagram:

.. image:: /images/Day3-entity_diagram.png

ストーリーで説明したカラムに加え、 created_at と updated_at のカラムを追加しました。
私たちは、オブジェクトが保存または更新されたときに自動的に値を設定するために Symfony を設定します。
In addition to the columns described in the stories, we have also added created_at and updated_at columns.
We will configure Symfony to set their value automatically when an object is saved or updated.

データベース
------------
The Database
------------

データベース内のジョブ、アフィリエイトとカテゴリを格納するために、Symfony 2.3.2 は Doctrine ORM を使用しています。
データベース接続パラメータを定義するには、（このチュートリアルでは、私たちは、MySQLを使用します） app/config/parameters.yml ファイルを編集する必要があります。
To store the jobs, affiliates and categories in the database, Symfony 2.3.2 uses Doctrine ORM. To define the database connection parameters, you have to edit the app/config/parameters.yml file (for this tutorial we will use MySQL):

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
Now that Doctrine knows about your database, you can have it create the database for you by typing the following command in your terminal:

.. code-block:: bash

    $ php app/console doctrine:database:create

スキーマ
--------
The Schema
----------

私たちのオブジェクトに関する教義を言うと、私たちは、オブジェクトをデータベースに格納する方法を説明します「メタデータ」ファイルが作成されます。今すぐあなたのコードエディタに移動し、SRC/ IBW/ JobeetBundle/リソース/ configディレクトリ内のディレクトリという教義を、作成してください。 Category.orm.yml、Job.orm.ymlとAffiliate.orm.yml：Doctrineは3つのファイルが含まれます。
To tell Doctrine about our objects, we will create “metadata” files that will describe how our objects will be stored in the database. Now go to your code editor and create a directory named doctrine, inside src/Ibw/JobeetBundle/Resources/config directory. Doctrine will contain three files: Category.orm.yml, Job.orm.yml and Affiliate.orm.yml.

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

今、Doctrineはコマンドを使用して私たちのために私たちのオブジェクトを定義するクラスを生成できます。
Now Doctrine can generate the classes that define our objects for us with the command:

.. code-block:: bash

    $ php app/console doctrine:generate:entities IbwJobeetBundle

Category.php、Job.phpとAffiliate.php：あなたはIbwJobeetBundleからエンティティディレクトリに見てみると、そこに新しく生成されたクラスがあります。オープンJob.phpとはcreated_atと以下のような値updated_atを設定します。
If you take a look into Entity directory from IbwJobeetBundle, you will find the newly generated classes in there: Category.php, Job.php and Affiliate.php. Open Job.php and set the created_at and updated_at values as below:

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

あなたはアフィリエイトクラスのはcreated_atの値のための同じをする。
You will do the same for created_at value of the Affiliate class:

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

これは、省またはオブジェクトを更新するときにDoctrineはcreated_atとを設定し、値updated_atするようになります。この動作は、上記のAffiliate.orm.ymlとJob.orm.ymlファイルで定義されていた。
また、以下のコマンドを使用して、私たちのデータベーステーブルを作成するためにDoctrineを聞いてきます。
This will make Doctrine to set the created_at and updated_at values when saving or updating objects. This behaviour was defined in the Affiliate.orm.yml and Job.orm.yml files listed above.
We will also ask Doctrine to create our database tables with the command below:

.. code-block:: bash

    $ php app/console doctrine:schema:update --force

.. note::

   このタスクは、開発時に使用されるべきである。体系的に本番データベースを更新するより堅牢な方法については、Doctrineのマイグレーションについて読ん。
   This task should only be used during the development. For a more robust method of systematically updating your production database, read about Doctrine migrations.

テーブルは、データベース内に作成されているが、データがそこにはありません。任意のWebアプリケーションでは、データの3つのタイプがあります。初期データ（これは仕事に適用するために必要とされ、私たちのケースでは私たちはいくつかの初期カテゴリと管理者ユーザを持つことになります）、テストデータ（アプリケーションをテストするために必要）と（アプリケーションの通常の寿命の間、ユーザーが作成した）ユーザーデータ。
いくつかの初期データに基づいてデータベースを作成するためには、`DoctrineFixturesBundle`_を使用します。セットアップこのバンドルには、次の手順に従ってくださいする必要があります。
The tables have been created in the database but there is no data in them. For any web application, there are three types of data: initial data (this is needed for the application to work, in our case we will have some initial categories and an admin user), test data (needed for the application to be tested) and user data (created by users during the normal life of the application).
To populate the database with some initial data, we will use `DoctrineFixturesBundle`_. To setup this bundle, we have to follow the next steps:

1が必要なセクションで、あなたのcomposer.jsonファイルに以下を追加します。
1. Add the following to your composer.json file, in the require section:

.. code-block:: json

   // ...
       "require": {
           // ...
           "doctrine/doctrine-fixtures-bundle": "dev-master",
           "doctrine/data-fixtures": "dev-master"
       },

   // ...

2ベンダライブラリを更新します。
2. Update the vendor libraries:

.. code-block:: bash

    $ php composer.phar update

3アプリ/ AppKernel.phpにバンドルDoctrineFixturesBundleを登録します。
3. Register the bundle DoctrineFixturesBundle in app/AppKernel.php:

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

今ではすべてがセットアップされ、私たちは新しいフォルダにデータをロードするためにいくつかの新しいクラスを作成しますが、名前のsrc / IBW/ JobeetBundle/ DataFixtures/ ORM、私たちのバンドル内：
Now that everything is set up, we will create some new classes to load data in a new folder, named src/Ibw/JobeetBundle/DataFixtures/ORM, in our bundle:

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

備品：あなたの備品が書き込まれたら、thedoctrineを使用して、コマンドラインを介してそれらをロードすることができloadコマンドを：
Once your fixtures have been written, you can load them via the command line by using thedoctrine:fixtures:load command:

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

使用しているデータベースをチェックすると今、あなたは、テーブルにロードされたデータが表示されるはずです。
Now, if you check your database, you should see the data loaded into tables.

See it in the browser
---------------------

あなたは下のコマンドを実行すると、新しいコントローラのsrc / IBW/ JobeetBundle/コントローラを作成します/リストのアクションの作成、編集、および削除ジョブ（およびそれに対応するテンプレート、フォームとルート）とJobController.php：
If you run the command below, it will create a new controller src/Ibw/JobeetBundle/Controllers/JobController.php with actions for listing, creating, editing and deleting jobs (and their corresponding templates, form and routes):

.. code-block:: bash

    $ php app/console doctrine:generate:crud --entity=IbwJobeetBundle:Job --route-prefix=ibw_job --with-write --format=yml

このコマンドを実行した後は、プロンプターをする必要があり、いくつかの設定を行う必要があります。だから彼らのために、デフォルトの答えを選択します。
ブラウザでこれを表示するには、私たちは、バンドルメインルーティングファイルにSRC/ IBW/ JobeetBundle/リソース/設定/ルーティング/ job.ymlで作成された新しいルートをインポートする必要があります。
After running this command, you will need to do some configurations the prompter requires you to. So just select the default answers for them.
To view this in the browser, we must import the new routes that were created in src/Ibw/JobeetBundle/Resources/config/routing/job.yml into our bundle main routing file:

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   IbwJobeetBundle_job:
           resource: "@IbwJobeetBundle/Resources/config/routing/job.yml"
           prefix:   /job
   # ...

また、編集ジョブフォームからドロップダウンカテゴリ別に使用するために私達のカテゴリクラスに_toString（）メソッドを追加する必要があります。
We will also need to add a _toString() method to our Category class to be used by the category drop down from the edit job form:

src/Ibw/JobeetBundle/Entity/Category.php

.. code-block:: php

   // ...

   public function __toString()
   {
       return $this->getName() ? $this->getName() : "";
   }

   // ...

キャッシュをクリアします。
Clear the cache:

.. code-block:: bash

    $ php app/console cache:clear --env=dev
    $ php app/console cache:clear --env=prod

//jobeet.local/app_dev.php/job/：開発環境、HTTPで、//jobeet.local/job/またはます。http：これで、ブラウザでジョブ·コントローラをテストすることができます。
You can now test the job controller in a browser: http://jobeet.local/job/ or, in development environment, http://jobeet.local/app_dev.php/job/ .

.. image:: /images/Day-3-index_page.png

これで、作成および編集の仕事ができます。必須フィールドを空白のままにするか、無効なデータを入力しようとしてみてください。そう、symfonyはデータベーススキーマをイントロスペクトすることにより、基本的なバリデーションルールを作成しました。
それだけだ。今日、私たちはかろうじてPHPコードを書かれているけど、微調整して、カスタマイズする準備ができて、仕事のモデルの作業Webモジュールを持っている。明日は、コントローラとビューに精通して取得します。次回お会いしましょう！
You can now create and edit jobs. Try to leave a required field blank, or try to enter invalid data. That’s right, Symfony has created basic validation rules by introspecting the database schema.
That’s all. Today, we have barely written PHP code but we have a working web module for the job model, ready to be tweaked and customized. Tomorrow, we will get familiar with the controller and the view. See you next time!

.. include:: common/license.rst.inc

.. _`DoctrineFixturesBundle`: http://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html