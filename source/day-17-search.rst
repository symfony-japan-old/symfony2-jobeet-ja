17日目: 検索
============

.. include:: common/original.rst.inc

| 14 日目では、最新の投稿で Jobeet ユーザーのジョブの更新を維持するため、いくつかのフィードを追加しました。
| 今日は検索エンジンという最新の機能を実装してユーザーエクスペリエンスを向上させます。

..
   In day 14, we added some feeds to keep Jobeet users up-to-date with new job posts.
   Today will help you to improve the user experience by implementing the latest main feature of the Jobeet website: the search engine.

テクノロジー
------------

| 今日は、 Jobeet に検索エンジンを追加いたします。有名な Java Lucene プロジェクトから移植された、 Zend Framework 提供の Zend Lucene と呼ばれるすばらしいライブラリを使用します。
| Jobeet のためのさらにもう1つの検索エンジンを作成するのは非常に複雑なタスクなため、代わりに、 Zend Lucene を使用します。
| 今日は、 Zend Lucene ライブラリのチュートリアルではなく、 それを Jobeet の Web サイトに統合する方法のチュートリアルです。
| より一般的には、サードパーティ製のライブラリをどのように symfony プロジェクトに統合するかのチュートリアルです。
| この技術についての詳細情報が必要な場合は、Zend Lucene のドキュメントを参照してください。

..
   Today, we want to add a search engine to Jobeet, and the Zend Framework provides a great library,
   called Zend Lucene, which is a port of the well-know Java Lucene project.
   Instead of creating yet another search engine for Jobeet, which is quite a complex task, we will use Zend Lucene.
   Today is not a tutorial about the Zend Lucene library, but how to integrate it into the Jobeet website;
   or more generally, how to integrate third-party libraries into a symfony project.
   If you want more information about this technology, please refer to the Zend Lucene documentation.

インストールと Zend Framework の設定
------------------------------------

| Zend Lucene ライブラリは、 Zend Framework の一部です。
| vendor/ ディレクトリに、 Symfony フレームワーク自身と一緒に、 Zend Framework をインストールするだけです。
| まず、Zend Framework をダウンロードし、 vendor/Zend/ ディレクトリを持つように、ファイルを解凍します。
| Zend Framework 2.* バージョンは Lucene ライブラリが統合されていないため、このチュートリアルでは、それらのいずれかもダウンロードしないことに注意してください。

..
   The Zend Lucene library is part of the Zend Framework.
   We will only install the Zend Framework into the vendor/ directory, alongside the symfony framework itself.
   First, download the Zend Framework and unzip the files so that you have a vendor/Zend/directory.
   Keep in mind that the 2.*  versions of Zend Framework do not have Lucene library integrated,
   so don’t download any of them for this tutorial.

.. note::

   以下の説明は、Zend Framework を1.12.3バージョンでテストされています。

   .. The following explanations have been tested with the 1.12.3 version of the Zend Framework.

.. image:: /images/Day-17-zend.jpg

次のファイルとディレクトリ以外のすべてを削除することで、ディレクトリをクリーンアップすることができます。

.. You can clean up the directory by removing everything but the following files and directories:

* Exception.php
* Loader/
* Loader.php
* Search/

その後、簡単に Zend autoloader を登録するため autoload.php ファイルに次のコードを追加します。

.. Then, add the following code to the autoload.php file to provide a simple way to register the Zend autoloader.

app/autoload.php

.. code-block:: php

   // ...

   set_include_path(__DIR__.'/../vendor'.PATH_SEPARATOR.get_include_path());
   require_once __DIR__.'/../vendor/Zend/Loader/Autoloader.php';
   Zend_Loader_Autoloader::getInstance();

   return $loader;

インデックス化
--------------

| Jobeet の検索エンジンは、ユーザーが入力したキーワードにマッチするすべての求人情報を返すことができるべきです。
| 何でも検索できるようになるには、ジョブに対して、Jobeet に対してインデックスが構築される必要があり、作成する新しいディレクトリ ( /web/data/ )に保存されます。
| Zend Lucene はインデックスがすでに存在するかどうかに応じて取得するために 2 つのメソッドを提供します。
| 既存のインデックスを返すか、もしくは、新しいインデックスを作成する、ジョブエンティティクラスのヘルパーメソッドを作成してみましょう。：

..
   The Jobeet search engine should be able to return all jobs matching keywords entered by the user.
   Before being able to search anything, an index has to be built for the jobs; for Jobeet, it will be stored in a new directory you will create, /web/data/ .
   Zend Lucene provides two methods to retrieve an index depending whether one already exists or not.
   Let’s create helper methods in the Job entity class that returns an existing index or creates a new one for us:

src/Ibw/JobeetBundle/Entity/Job.php

.. code-block:: php

   // ...

   class Job
   {
       // ...

       static public function getLuceneIndex()
       {
           if (file_exists($index = self::getLuceneIndexFile())) {
               return \Zend_Search_Lucene::open($index);
           }

           return \Zend_Search_Lucene::create($index);
       }

       static public function getLuceneIndexFile()
       {
           return __DIR__.'/../../../../web/data/job.index';
       }
   }

| ジョブが作成または更新されるたびに、インデックスを更新する必要があります。
| ジョブがデータベースにシリアライズされるたびにインデックスを更新するように ORM ファイルを編集します。

..
   Each time a job is created or updated, the index must be updated.
   Edit the ORM file to update the index whenever a job is serialized to the database:

src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml

.. code-block:: yaml

   # ...
       # ...
       lifecycleCallbacks:
           # ...
           postPersist: [ upload, updateLuceneIndex ]
           postUpdate: [ upload, updateLuceneIndex ]
           # ...

ここで、 ``generate:entities`` コマンド実行し、 Doctrineによって updateLuceneIndex() メソッドがジョブ·クラスの内部に生成されるようにします。

.. Now, run the generate:entities command, so that the updateLuceneIndex() method to be generated by Doctrine inside the Job class:

.. code-block:: bash

   $ php app/console doctrine:generate:entities IbwJobeetBundle

次に、実際の作業を行う updateLuceneIndex() メソッドを編集します。

src/Ibw/JobeetBundle/Entity/Job.php

.. code-block:: php

   class Job
   {
       // ...

       public function updateLuceneIndex()
       {
           $index = self::getLuceneIndex();

           // remove existing entries
           foreach ($index->find('pk:'.$this->getId()) as $hit)
           {
             $index->delete($hit->id);
           }

           // don't index expired and non-activated jobs
           if ($this->isExpired() || !$this->getIsActivated())
           {
             return;
           }

           $doc = new \Zend_Search_Lucene_Document();

           // store job primary key to identify it in the search results
           $doc->addField(\Zend_Search_Lucene_Field::Keyword('pk', $this->getId()));

           // index job fields
           $doc->addField(\Zend_Search_Lucene_Field::UnStored('position', $this->getPosition(), 'utf-8'));
           $doc->addField(\Zend_Search_Lucene_Field::UnStored('company', $this->getCompany(), 'utf-8'));
           $doc->addField(\Zend_Search_Lucene_Field::UnStored('location', $this->getLocation(), 'utf-8'));
           $doc->addField(\Zend_Search_Lucene_Field::UnStored('description', $this->getDescription(), 'utf-8'));

           // add job to the index
           $index->addDocument($doc);
           $index->commit();
       }
   }

| Zend Lucene は既存エントリを更新することができないため、インデックス内にジョブがすでに存在する場合は、はじめに削除します。
| ジョブ自体のインデックス作成は簡単です。主キーはジョブ検索するときに将来の参照用に保存されます。
| メインカラム（位置、会社、場所、および説明）はインデックス化されますが、表示する際に本物のオブジェクトを使うのでインデックスに格納はされません。
| また、削除されたジョブエントリをインデックスから削除するため、 deleteLuceneIndex() メソッドを作成する必要があります。
| 更新と同様の処理を削除でも行います。
| ORM ファイルの postremove セクションに deleteLuceneIndex() メソッドを追加します。

..
   As Zend Lucene is not able to update an existing entry, it is removed first if the job already exists in the index.
   Indexing the job itself is simple: the primary key is stored for future reference when searching jobs
   and the main columns (position, company, location, and description) are indexed but not stored in the index as we will use the real objects to display the results.
   We also need create a deleteLuceneIndex() method to remove the entry of the deleted job from the index.
   As we did for the update, we will do for delete.
   Start by adding the deleteLuceneIndex() method to postRemove section of the ORM file:

src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml

.. code-block:: yaml

   # ...
       # ...
       lifecycleCallbacks:
           # ...
           postRemove: [ removeUpload, deleteLuceneIndex ]

| 再度、エンティティを生成するコマンドを実行します。
| ここで、エンティティファイルに移動し、 deleteLuceneIndex() メソッドを実装します。

..
   Again, run the command for generating entities.
   Now, go to entity file and implement the deleteLuceneIndex() method:

src/Ibw/JobeetBundle/Entity/Job.php

.. code-block:: php

   class Job
   {
       // ...

       public function deleteLuceneIndex()
       {
           $index = self::getLuceneIndex();

           foreach ($index->find('pk:'.$this->getId()) as $hit) {
               $index->delete($hit->id);
           }
       }
   }

インデックスはコマンドラインからもWebからも更新されるので、構成に応じてインデックディレクトリのパーミッションを変更する必要があります。

.. As the index is modified from the command line and also from the web, you must change the index directory permissions accordingly depending on your configuration:

.. code-block:: bash

   $ chmod -R 777 web/data

これで、すべてそろいましたので、インデックス対象のフィクスチャーデータをリロードします。

.. Now that we have everything in place, you can reload the fixture data to index them:

.. code-block:: bash

   $ php app/console doctrine:fixtures:load

検索
----

検索を実装することは、とても簡単です。まず、ルートを作成します。

src/Ibw/JobeetBundle/Resources/config/routing/job.yml

.. code-block:: yaml

   # ...

   ibw_job_search:
       pattern: /search
       defaults: { _controller: "IbwJobeetBundle:Job:search" }

そして、対応するアクション。

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Controller;

   use Symfony\Component\HttpFoundation\Request;
   use Symfony\Bundle\FrameworkBundle\Controller\Controller;
   use Ibw\JobeetBundle\Entity\Job;
   use Ibw\JobeetBundle\Form\JobType;

   class JobController extends Controller
   {
       // ...

       public function searchAction(Request $request)
       {
           $em = $this->getDoctrine()->getManager();
           $query = $this->getRequest()->get('query');

           if(!$query) {
               return $this->redirect($this->generateUrl('ibw_job'));
           }

           $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery($query);

           return $this->render('IbwJobeetBundle:Job:search.html.twig', array('jobs' => $jobs));
       }
   }

| searchAction() の内部では、ユーザーは、クエリリクエストが存在しないか空の場合は、 JobController の index アクションに転送されます。
| テンプレートには、非常に簡単です。

..
   Inside the searchAction(), the user is forwarded to the index action of the JobController if the query request does not exist or is empty.
   The template is also quite straightforward:

src/Ibw/JobeetBundle/Resources/views/Job/search.html.twig

.. code-block:: html+jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/jobs.css') }}" type="text/css" media="all" />
   {% endblock %}

   {% block content %}
       <div id="jobs">
           {% include 'IbwJobeetBundle:Job:list.html.twig' with {'jobs': jobs} %}
       </div>
   {% endblock %}

検索自体は getForLuceneQuery() メソッドに委譲されています。

.. The search itself is delegated to the getForLuceneQuery() method:

src/Ibw/JobeetBundle/Repository/JobRepository.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Repository;

   use Doctrine\ORM\EntityRepository;
   use Ibw\JobeetBundle\Entity\Job;

   class JobRepository extends EntityRepository
   {
       // ...

       public function getForLuceneQuery($query)
       {
           $hits = Job::getLuceneIndex()->find($query);

           $pks = array();
           foreach ($hits as $hit)
           {
             $pks[] = $hit->pk;
           }

           if (empty($pks))
           {
             return array();
           }

           $q = $this->createQueryBuilder('j')
               ->where('j.id IN (:pks)')
               ->setParameter('pks', $pks)
               ->andWhere('j.is_activated = :active')
               ->setParameter('active', 1)
               ->setMaxResults(20)
               ->getQuery();

           return $q->getResult();
       }
   }

| Lucene インデックスからすべての結果を取得した後、非アクティブなジョブをフィルタリングし、結果の数を 20 に制限します。
| 動作させるためにレイアウトを更新します。

..
   After we get all results from the Lucene index, we filter out the inactive jobs, and limit the number of results to 20.
   To make it work, update the layout:

.. code-block:: html+jinja

   <!-- ... -->
       <!-- ... -->
           <div class="search">
               <h2>Ask for a job</h2>
               <form action="{{ path('ibw_job_search') }}" method="get">
                   <input type="text" name="query" value="{{ app.request.get('query') }}" id="search_keywords" />
                   <input type="submit" value="search" />
                   <div class="help">
                       Enter some keywords (city, country, position, ...)
                   </div>
               </form>
           </div>
       <!-- ... -->
   <!-- ... -->

単体テスト
----------

| 検索エンジンをテストするためにどのような単体テストを作成すべきでしょうか？
| 明らかに Zend Lucene ライブラリそのもののテストではなく、ジョブ·クラスとの統合をテストすることです。
| JobRepositoryTest.php ファイルの最後に次のテストを追加します。

..
   What kind of unit tests do we need to create to test the search engine?
   We obviously won’t test the Zend Lucene library itself, but its integration with the Job class.
   Add the following test at the end of the JobRepositoryTest.php file:

src/Ibw/JobeetBundle/Tests/Repository/JobRepositoryTest.php

.. code-block:: php

   // ...
   use Ibw\JobeetBundle\Entity\Job;

   class JobRepositoryTest extends WebTestCase
   {
       // ...

       public function testGetForLuceneQuery()
       {
           $em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           $job = new Job();
           $job->setType('part-time');
           $job->setCompany('Sensio');
           $job->setPosition('FOO6');
           $job->setLocation('Paris');
           $job->setDescription('WebDevelopment');
           $job->setHowToApply('Send resumee');
           $job->setEmail('jobeet@example.com');
           $job->setUrl('http://sensio-labs.com');
           $job->setIsActivated(false);

           $em->persist($job);
           $em->flush();

           $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery('FOO6');
           $this->assertEquals(count($jobs), 0);

           $job = new Job();
           $job->setType('part-time');
           $job->setCompany('Sensio');
           $job->setPosition('FOO7');
           $job->setLocation('Paris');
           $job->setDescription('WebDevelopment');
           $job->setHowToApply('Send resumee');
           $job->setEmail('jobeet@example.com');
           $job->setUrl('http://sensio-labs.com');
           $job->setIsActivated(true);

           $em->persist($job);
           $em->flush();

           $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery('position:FOO7');
           $this->assertEquals(count($jobs), 1);
           foreach ($jobs as $job_rep) {
               $this->assertEquals($job_rep->getId(), $job->getId());
           }

           $em->remove($job);
           $em->flush();

           $jobs = $em->getRepository('IbwJobeetBundle:Job')->getForLuceneQuery('position:FOO7');

           $this->assertEquals(count($jobs), 0);
       }
   }

| 非アクティブなジョブ、または、削除されたジョブは、検索結果に表示されないことをテストします。
| 与えられた条件に一致するジョブが結果に表示されることも確認してください。

..
   We test that a non activated job, or a deleted one does not show up in the search results;
   we also check that jobs matching the given criteria do show up in the results.

タスク
------

最終的には、 JobeetCleanup タスクを更新して、古いエントリ（たとえばジョブが期限切れになったときに）のインデックスをクリーンアップし、随時インデックスを最適化する必要があります。

.. Eventually, we need to update the JobeetCleanup task to cleanup the index from stale entries (when a job expires for example) and optimize the index from time to time:

src/Ibw/JobeetBundle/Command/JobeetCleanupCommand.php

.. code-block:: php

   // ...
   use  Ibw\JobeetBundle\Entity\Job;

   class JobeetCleanupCommand extends ContainerAwareCommand
   {
       // ...

       protected function execute(InputInterface $input, OutputInterface $output)
       {
           $days = $input->getArgument('days');

           $em = $this->getContainer()->get('doctrine')->getManager();

           // cleanup Lucene index
           $index = Job::getLuceneIndex();

           $q = $em->getRepository('IbwJobeetBundle:Job')->createQueryBuilder('j')
             ->where('j.expires_at < :date')
             ->setParameter('date',date('Y-m-d'))
             ->getQuery();

           $jobs = $q->getResult();
           foreach ($jobs as $job)
           {
             if ($hit = $index->find('pk:'.$job->getId()))
             {
               $index->delete($hit->id);
             }
           }

           $index->optimize();

           $output->writeln('Cleaned up and optimized the job index');

           // Remove stale jobs
           $nb = $em->getRepository('IbwJobeetBundle:Job')->cleanup($days);

           $output->writeln(sprintf('Removed %d stale jobs', $nb));
       }
   }

| このタスクはインデックスからすべての期限切れのジョブを削除し、Zend Lucene の組み込みの optimize() メソッドによって最適化します。
| 今日は、 1 時間も経たないうちに、多くの機能を備えた完全な検索エンジンを実装しました。
| プロジェクトに新しい機能を追加したいときはいつも、まだどこかに解決されていないものがあるか確認してください。
| 明日は、検索ボックスにユーザーの種類などのリアルタイムで結果を更新することで、
| 検索エンジンのレスポンスを強化するために慎ましく JavaScript を使います。
| もちろん、これは Symfony で AJAX を使用する方法について話をすることになります。

..
   The task removes all expired jobs from the index and then optimizes it thanks to the Zend Lucene built-in optimize() method.
   Along this day, we implemented a full search engine with many features in less than an hour.
   Every time you want to add a new feature to your projects, check that it has not yet been solved somewhere else.
   Tomorrow we will use some unobtrusive JavaScripts to enhance the responsiveness of the search engine by updating the results in real-time as the user types in the search box.
   Of course, this will be the occasion to talk about how to use AJAX with symfony.

.. include:: common/license.rst.inc
