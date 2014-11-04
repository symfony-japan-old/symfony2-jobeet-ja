6日目: モデルの続き
================

.. include:: common/original.rst.inc

Doctrine クエリーオブジェクト
-----------------------------

| 2 日目の要件として、「ホームページでは、ユーザーは最新の有効なジョブを閲覧します」としました。
| しかし、現在はすべてのジョブが、アクティブであるかに関わらず表示されています。


src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

   class JobController extends Controller
   {
       public function indexAction()
       {
           $em = $this->getDoctrine()->getManager();

           $entities = $em->getRepository('IbwJobeetBundle:Job')->findAll();

           return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
               'entities' => $entities
           ));

    // ...
   }

| アクティブなジョブとは、 30 日前以内に投稿されたものです。
| ``$entities = $em->getRepository('IbwJobeetBundle')->findAll()`` メソッドは、すべてのジョブを取得するために、
| データベースへのリクエストを行います。
| 現在は条件をなにも指定していません。そのため、すべてのレコードがデータベースから取り出されてしまいます。
| では、アクティブなジョブのみを選択するよう変更してみましょう。


.. code-block:: php

   public function indexAction()
   {
       $em = $this->getDoctrine()->getManager();

       $query = $em->createQuery(
           'SELECT j FROM IbwJobeetBundle:Job j WHERE j.created_at > :date'
       )->setParameter('date', date('Y-m-d H:i:s', time() - 86400 * 30));
       $entities = $query->getResult();

       return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
           'entities' => $entities
       ));
   }

Doctrine が生成した SQL のデバッグ
----------------------------------

| デバッグツールバーは、例えば、期待通りに動作しないクエリをデバッグするときに、
| Doctrine によって生成された SQL を参照する大きな助けとなります。
| dev 環境では、Symfony は Web デバッグツールバーのおかげで、
| 必要なすべての情報がブラウザ上で快適に利用できます（ http：//jobeet.local/app_dev.php ）。

.. image:: /images/Day-6-web-debug-toolbar.png

オブジェクトのシリアライズ
--------------------------

| 上記のコードは動作していても、 2 日目の「ユーザーは戻ってきて再度アクティブにするか、
| ジョブの期間の検証を 30 日以上に広げることが出来、、、」という要件から考慮すると、完璧にはほど遠いです。
| 上記のコードは created_at の値にのみ依存しており、このカラムには作成日を格納しているため、要件を満しません。
| 3 日目に説明したデータベーススキーマを覚えていれば、 expires_at カラムも定義したことを覚えているでしょう。
| この値は、フィクスチャーファイルに設定されていない場合、空のままです。
| しかし、ジョブが作成されるときに、自動的に現在の日付から 30 日後に設定することができます。
| Doctrine のオブジェクトがデータベースにシリアライズされる前に自動的に何かをする必要があるときは、
| 先ほど created_at カラムで行ったように、オブジェクトをデータベースにマッピングするファイルに、
| ライフサイクルコールバックの新しいアクションを追加することで可能です。
Even if the code above works, it is far from perfect as it does not take into account some requirements from Day 2:
“A user can come back to re-activate or extend the validity of the job for an extra 30 days..”.
But as the above code only relies on the created_at value, and because this column stores the creation date, we cannot satisfy the above requirement.
If you remember the database schema we have described during Day 3, we also have defined an expires_at column.
Currently, if this value is not set in fixture file, it remains always empty.
But when a job is created, it can be automatically set to 30 days after the current date.
When you need to do something automatically before a Doctrine object is serialized to the database,
you can add a new action to the lifecycle callbacks in the file that maps objects to the database, like we did earlier for the created_at column:

src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orml.yml

.. code-block:: yaml

   # ...
       # ...
       lifecycleCallbacks:
           prePersist: [ setCreatedAtValue, setExpiresAtValue ]
           preUpdate: [ setUpdatedAtValue ]

Doctrine に新しい関数が追加されるので、エンティティクラスを再生成する必要があります。

.. code-block:: bash

   $ php app/console doctrine:generate:entities IbwJobeetBundle

src/Ibw/JobeetBundle/Entity/Job.php ファイルを開き、新たなメソッドを追加します。

src/Ibw/JobeetBundle/Entity/Job.php

.. code-block:: php

   // ...

   class Job
   {
       // ...

       public function setExpiresAtValue()
       {
           if(!$this->getExpiresAt()) {
               $now = $this->getCreatedAt() ? $this->getCreatedAt()->format('U') : time();
               $this->expires_at = new \DateTime(date('Y-m-d H:i:s', $now + 86400 * 30));
           }
       }
   }

では、有効なジョブを選択するためには created_at の代わりに expires_at カラムを使用するようにアクションを変更してみましょう。

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

       public function indexAction()
       {
           $em = $this->getDoctrine()->getManager();

           $query = $em->createQuery(
               'SELECT j FROM IbwJobeetBundle:Job j WHERE j.expires_at > :date'
       )->setParameter('date', date('Y-m-d H:i:s', time()));
           $entities = $query->getResult();

           return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
               'entities' => $entities
           ));
       }

   // ...

フィクスチャーに追加
--------------------

| データベース内のジョブが数日前に投稿されたばかりのため、ブラウザで Jobeet のホームページをリフレッシュしても何も変わりません。
| それでは、期限切れのジョブ情報をフィクスチャーに追加しましょう。

src/Ibw/JobeetBundle/DataFixtures/ORM/LoadJobData.php

.. code-block:: php

   // ...

       public function load(ObjectManager $em)
       {
           $job_expired = new Job();
           $job_expired->setCategory($em->merge($this->getReference('category-programming')));
           $job_expired->setType('full-time');
           $job_expired->setCompany('Sensio Labs');
           $job_expired->setLogo('sensio-labs.gif');
           $job_expired->setUrl('http://www.sensiolabs.com/');
           $job_expired->setPosition('Web Developer Expired');
           $job_expired->setLocation('Paris, France');
           $job_expired->setDescription('Lorem ipsum dolor sit amet, consectetur adipisicing elit.');
           $job_expired->setHowToApply('Send your resume to lorem.ipsum [at] dolor.sit');
           $job_expired->setIsPublic(true);
           $job_expired->setIsActivated(true);
           $job_expired->setToken('job_expired');
           $job_expired->setEmail('job@example.com');
           $job_expired->setCreatedAt(new \DateTime('2005-12-01'));

           // ...

           $em->persist($job_expired);
           // ...
       }

   // ...

フィクスチャーをリロードし、古いジョブが表示されないことを確認するためにブラウザをリフレッシュします。

.. code-block:: bash

   $ php app/console doctrine:fixtures:load

リファクタリング
----------------

| 書いたコードは動作しますが、まだ正しくありません。問題を発見することはできますでしょうか？
| Doctrine のクエリコードはアクション（コントローラ層）に属さず、 Model レイヤーに所属します。
| MVC モデルでは、モデルはすべてのビジネスロジックを定義し、 Controller はデータを読み取るモデルのみを呼び出します。
| コードは、ジョブのコレクションを返すように、そのコードをモデルに移動しましょう​​。
|  そのためには、ジョブのエンティティのカスタムリポジトリクラスを作成し、そのクラスにクエリを追加する必要があります。
| /src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml を開いて、そこに次の行を追加します。
Although the code we have written works fine, it’s not quite right yet. Can you spot the problem?
The Doctrine query code does not belong to the action (the Controller layer), it belongs to the Model layer.
In the MVC model, the Model defines all the business logic, and the Controller only calls the Model to retrieve data from it.
As the code returns a collection of jobs, let’s move the code to the model.
For that we will need to create a custom repository class for Job entity and to add the query to that class.
Open /src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml and add the following to it:

src/Ibw/JobeetBundle/Resources/config/doctrine/Job.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\Job:
       type: entity
       repositoryClass: Ibw\JobeetBundle\Repository\JobRepository
       # ...

Doctrine は、以前に使用した ``generate:entities`` コマンドを​実行することで、リポジトリクラスを生成することができます。

.. code-block:: bash

   $ php app/console doctrine:generate:entities IbwJobeetBundle

| 次に、新規に生成されたリポジトリクラスに新しいメソッド - getActiveJobs() - を追加します。
| このメソッドは、 expires_at のカラムでソートし、有効なすべてのジョブのエンティティを照会します。
| さらに、$category_id パラメータを受信した場合、カテゴリでフィルタリングしたエンティティを照会します。

src/Ibw/JobeetBundle/Repository/JobRepository.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Repository;

   use Doctrine\ORM\EntityRepository;

   /**
    * JobRepository
    *
    * This class was generated by the Doctrine ORM. Add your own custom
    * repository methods below.
    */
   class JobRepository extends EntityRepository
   {
       public function getActiveJobs($category_id = null)
       {
           $qb = $this->createQueryBuilder('j')
               ->where('j.expires_at > :date')
               ->setParameter('date', date('Y-m-d H:i:s', time()))
               ->orderBy('j.expires_at', 'DESC');

           if($category_id)
           {
               $qb->andWhere('j.category = :category_id')
                   ->setParameter('category_id', $category_id);
           }

           $query = $qb->getQuery();

           return $query->getResult();
       }
   }

ここで、アクションのなかで、アクティブなジョブを取得するために上記の新しいメソッドを使用します。

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

       public function indexAction()
       {
           $em = $this->getDoctrine()->getManager();

           $entities = $em->getRepository('IbwJobeetBundle:Job')->getActiveJobs();

           return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
               'entities' => $entities
           ));
       }

   // ...

* このリファクタリングは、前のコードに比べていくつかの利点があります。
* 有効なジョブを取得するためのロジックは、モデルに属します。
* コントローラーのコードは、より薄く、より読みやすくします。
* getActiveJobs() メソッドは、再利用可能なされている（たとえば別のアクションで）。
* 現在、モデルコードは単体テスト（ユニットテスト）が可能です。

* This refactoring has several benefits over the previous code:
* The logic to get the active jobs is now in the Model, where it belongs
* The code in the controller is thinner and much more readable
* The getActiveJobs() method is re-usable (for instance in another action)
* The model code is now unit testable

ホームページのカテゴリー
------------------------

| 二日目の要件によると、カテゴリでソートされたジョブを持っていなくてはいけません。
| 今まで、ジョブのカテゴリーを考慮していませんでした。要件からはホームページでカテゴリに基づいて表示しなければなりません。
| まず、少なくとも1つの有効なジョブからすべてのカテゴリを取得する必要があります。
| ジョブクラスに行ったように、カテゴリエンティティのリポジトリクラスを作成します。
According to the second day’s requirements we need to have jobs sorted by categories.
Until now, we have not taken the job category into account. From the requirements, the homepage must display jobs by category.
First, we need to get all categories with at least one active job.
Create a repository class for the Category entity like we did for Job:

src/Ibw/JobeetBundle/Resources/config/doctrine/Category.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\Category:
       type: entity
       repositoryClass: Ibw\JobeetBundle\Repository\CategoryRepository
       #...

リポジトリクラスを生成します。

.. code-block:: bash

   $ php app/console doctrine:generate:entities IbwJobeetBundle

CategoryRepository クラスを開き、 getWithJobs() メソッドを追加します。

src/Ibw/JobeetBundle/Repository/CategoryRepository.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Repository;

   use Doctrine\ORM\EntityRepository;

   /**
    * CategoryRepository
    *
    * This class was generated by the Doctrine ORM. Add your own custom
    * repository methods below.
    */
   class CategoryRepository extends EntityRepository
   {
       public function getWithJobs()
       {
           $query = $this->getEntityManager()->createQuery(
               'SELECT c FROM IbwJobeetBundle:Category c LEFT JOIN c.jobs j WHERE j.expires_at > :date'
           )->setParameter('date', date('Y-m-d H:i:s', time()));

           return $query->getResult();
       }
   }

それに応じて index アクションを変更します。

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

       public function indexAction()
       {
           $em = $this->getDoctrine()->getManager();

           $categories = $em->getRepository('IbwJobeetBundle:Category')->getWithJobs();

           foreach($categories as $category) {
               $category->setActiveJobs($em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId()));
           }

           return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
               'categories' => $categories
           ));
       }

   // ...

これが機能するためには、 Category クラスに新しいプロパティ active_jobs を追加する必要があります。

src/Ibw/JobeetBundle/Entity/Category.php

.. code-block:: php

   class Category
   {
       // ...

       private $active_jobs;

       // ...

       public function setActiveJobs($jobs)
       {
           $this->active_jobs = $jobs;
       }

       public function getActiveJobs()
       {
           return $this->active_jobs;
       }
   }

テンプレートでは、すべてのカテゴリを反復処理し、有効なジョブを表示する必要があります。

src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig

.. code-block:: html+jinja

   <!-- ... -->
   {% block content %}
       <div id="jobs">
           {% for category in categories %}
               <div>
                   <div class="category">
                       <div class="feed">
                           <a href="">Feed</a>
                       </div>
                       <h1>{{ category.name }}</h1>
                   </div>
                   <table class="jobs">
                       {% for entity in category.activejobs %}
                           <tr class="{{ cycle(['even', 'odd'], loop.index) }}">
                               <td class="location">{{ entity.location }}</td>
                               <td class="position">
                                   <a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug }) }}">
                                       {{ entity.position }}
                                   </a>
                               </td>
                                <td class="company">{{ entity.company }}</td>
                           </tr>
                       {% endfor %}
                   </table>
               </div>
           {% endfor %}
       </div>
   {% endblock %}

結果を制限
----------

| ホームページのジョブリストのために実装すべき要件がまだ1つあります。
| それは、ジョブリストの数を 10 個に制限することです。
| JobRepository:: getActiveJobs() メソッドに $max のパラメータを追加するだけで十分です。

src/Ibw/JobeetBundle/Repository/JobRepository.php

.. code-block:: php

    public function getActiveJobs($category_id = null, $max = null)
    {
        $qb = $this->createQueryBuilder('j')
            ->where('j.expires_at > :date')
            ->setParameter('date', date('Y-m-d H:i:s', time()))
            ->orderBy('j.expires_at', 'DESC');

        if($max) {
            $qb->setMaxResults($max);
        }

        if($category_id) {
            $qb->andWhere('j.category = :category_id')
                ->setParameter('category_id', $category_id);
        }

        $query = $qb->getQuery();

        return $query->getResult();
    }

getActiveJobs() メソッドの呼び出しのパラメータに $max が含まれるように変更します。

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

       public function indexAction()
       {
           $em = $this->getDoctrine()->getManager();

           $categories = $em->getRepository('IbwJobeetBundle:Category')->getWithJobs();

           foreach($categories as $category)
           {
               $category->setActiveJobs($em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId(), 10));
           }

           return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
               'categories' => $categories
           ));
       }

   // ...

カスタム設定
------------

| JobController の indexAction メソッドでは、カテゴリごとの最大ジョブ数は、ハードコーディングされていました。
| その10個の制限は、設定可能にする方がよいです。
| Symfony の  app/config/config.yml ファイルの ``parameters`` キーの下に（ ``parameters`` が存在しない場合は作成して）
| アプリケーション用のカスタムパラメータを定義できます。
In the JobController, indexAction method, we have hardcoded the number of max jobs returned for a category.
It would have been better to make the 10 limit configurable.
In Symfony, you can define custom parameters for your application in the app/config/config.yml file, under the parameters key (if the parameters key doesn’t exist, create it):

app/config/config.yml

.. code-block:: yaml

   # ...

   parameters:
       max_jobs_on_homepage: 10

これでコントローラからパラメーターにアクセスできます。

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

       public function indexAction()
       {
           $em = $this->getDoctrine()->getManager();

           $categories = $em->getRepository('IbwJobeetBundle:Category')->getWithJobs();

           foreach($categories as $category) {
               $category->setActiveJobs($em->getRepository('IbwJobeetBundle:Job')->getActiveJobs($category->getId(), $this->container->getParameter('max_jobs_on_homepage')));
           }

           return $this->render('IbwJobeetBundle:Job:index.html.twig', array(
               'categories' => $categories
           ));
       }

   // ...

動的なフィクスチャー
--------------------

| まだ、データベースにあるジョブが非常に少ないため、何の違いも表示されません。
| フィクスチャーにまとまったジョブを追加する必要があります。
| そのために手動で既存のジョブの 10 または 20 倍をコピーアンドペーストする...というよりももっと良い方法があります。
| フィクスチャーファイルにおいても重複することは悪いことです。
For now, you won’t see any difference because we have a very small amount of jobs in our database. We need to add a bunch of jobs to the fixture.
So, you can copy and paste an existing job ten or twenty times by hand… but there’s a better way. Duplication is bad, even in fixture files:

src/Ibw/JobeetBundle/DataFixtures/ORM/LoadJobData.php

.. code-block:: php

   // ...

   public function load(ObjectManager $em)
   {
       // ...

       for($i = 100; $i <= 130; $i++)
       {
           $job = new Job();
           $job->setCategory($em->merge($this->getReference('category-programming')));
           $job->setType('full-time');
           $job->setCompany('Company '.$i);
           $job->setPosition('Web Developer');
           $job->setLocation('Paris, France');
           $job->setDescription('Lorem ipsum dolor sit amet, consectetur adipisicing elit.');
           $job->setHowToApply('Send your resume to lorem.ipsum [at] dolor.sit');
           $job->setIsPublic(true);
           $job->setIsActivated(true);
           $job->setToken('job_'.$i);
           $job->setEmail('job@example.com');

           $em->persist($job);
       }

       // ...
       $em->flush();
   }

   // ...

| ``doctrine:fixtures:load`` タスクでフィクスチャーを再読み込みできます。
| ホームページのプログラミングカテゴリに 10 個のジョブだけが表示されます。

.. image:: /images/Day-6-limited-no-of-jobs.png

ジョブページを安全にする
----------------------

| ジョブが終了したときは、URLを知っていても、もうそれにアクセスすることはできません。
| 期限切れのジョブ用のURLを、 ``SELECT id, token FROM job WHERE expires_at < NOW()`` で調べて、
| データベース内の実際の id で URL の ID を置き換えて試してみてください。
| /app_dev.php/job/sensio-labs/paris-france/ID/web-developer-expired
| ジョブを表示する代わりに、404ページにユーザーを転送する必要があります。
| このために JobRepository クラスに新しい関数を作成します。

When a job expires, even if you know the URL, it must not be possible to access it anymore.
Try the URL for the expired job (replace the id with the actual id in your database – SELECT id, token FROM job WHERE expires_at < NOW()):
/app_dev.php/job/sensio-labs/paris-france/ID/web-developer-expired
Instead of displaying the job, we need to forward the user to a 404 page.
For this we will create a new function in the JobRepository:

src/Ibw/JobeetBundle/Repository/JobRepository.php

.. code-block:: php

   // ...

       public function getActiveJob($id)
       {
           $query = $this->createQueryBuilder('j')
               ->where('j.id = :id')
               ->setParameter('id', $id)
               ->andWhere('j.expires_at > :date')
               ->setParameter('date', date('Y-m-d H:i:s', time()))
               ->setMaxResults(1)
               ->getQuery();

           try {
               $job = $query->getSingleResult();
           } catch (\Doctrine\Orm\NoResultException $e) {
               $job = null;
           }

           return $job;
       }

| getSingleResult() メソッドは、結果が返されない場合には ``Doctrine\ORM\NoResultException`` 例外がスローされます。
| また、複数の結果が返された場合は、 ``Doctrine\ORM\NonUniqueResultException`` 例外がスローされます。
| この方法を使用する場合は、try-catch ブロックで囲んで、結果がひとつだけ返されることを保証する必要があるかもしれません。
| 今すぐ新しいリポジトリメソッドを使用するように JobController クラスの showAction() メソッドを変更します。
The getSingleResult() method throws a Doctrine\ORM\NoResultException exception if no results are returned and
a Doctrine\ORM\NonUniqueResultException if more than one result is returned.
If you use this method, you may need to wrap it in a try-catch block and ensure that only one result is returned.
Now change the showAction() from the JobController to use the new repository method:

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

   $entity = $em->getRepository('IbwJobeetBundle:Job')->getActiveJob($id);

   // ...

期限切れのジョブを取得しようとした場合、404ページに転送されます。

.. image:: /images/Day-6-no-job-found.png

これで今日のすべてです！明日またお会いしましょう。カテゴリページを作っていきます。

.. include:: common/license.rst.inc
