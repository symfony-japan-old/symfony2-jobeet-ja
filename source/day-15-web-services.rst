15日目: Web Services
====================
Day 15: Web Services
====================

.. include:: common/original.rst.inc

Jobeetの上のフィードを追加することで、求職者はリアルタイムで新しい求人を知ることができる。 
あなたが仕事を投稿するときフェンスの向こう側では、あなたは、可能なエクスポージャーを持つことになるでしょう。あなたの仕事は、小さな多くのウェブサイトにシンジケートされている場合は、右の人を見つける機会が増えます。つまり、ロングテールの力です。アフィリエイトは、私たちが今日開発するWebサービスに自分のウェブサイト上のおかげで投稿された最新の求人情報を公開することができるようになります。
With the addition of feeds on Jobeet, job seekers can now be informed of new jobs in real-time.
On the other side of the fence, when you post a job, you will want to have the greatest exposure possible. If your job is syndicated on a lot of small websites, you will have a better chance to find the right person. That’s the power of the long tail. Affiliates will be able to publish the latest posted jobs on their websites thanks to the web services we will develop today.

Affiliates
----------

私たちはすでにこのチュートリアルの2日目で言ったように、アフィリエイトは、現在アクティブなジョブリストを取得します。
As we already said in day 2 of this tutorial, an affiliate retrieves the current active job list.

The fixtures
------------

それではアフィリエイト用に新しいフィクスチャファイルを作成してみましょう：
Let’s create a new fixture file for the affiliates:

.. code-block:: php

   src/Ibw/JobeetBundle/DataFixtures/ORM/LoadAffiliateData.php

   namespace Ibw\JobeetBundle\DataFixtures\ORM;

   use Doctrine\Common\Persistence\ObjectManager;
   use Doctrine\Common\DataFixtures\AbstractFixture;
   use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
   use Ibw\JobeetBundle\Entity\Affiliate;

   class LoadAffiliateData extends AbstractFixture implements OrderedFixtureInterface
   {
       public function load(ObjectManager $em)
       {
           $affiliate = new Affiliate();

           $affiliate->setUrl('http://sensio-labs.com/');
           $affiliate->setEmail('address1@example.com');
           $affiliate->setToken('sensio-labs');
           $affiliate->setIsActive(true);
           $affiliate->addCategorie($em->merge($this->getReference('category-programming')));

           $em->persist($affiliate);

           $affiliate = new Affiliate();

           $affiliate->setUrl('/');
           $affiliate->setEmail('address2@example.org');
           $affiliate->setToken('symfony');
           $affiliate->setIsActive(false);
           $affiliate->addCategorie($em->merge($this->getReference('category-programming')), $em->merge($this->getReference('category-design')));

           $em->persist($affiliate);
           $em->flush();

           $this->addReference('affiliate', $affiliate);
       }

       public function getOrder()
       {
           return 3; // This represents the order in which fixtures will be loaded
       }
   }

今、ちょうど次のコマンドを実行し、あなたのフィクスチャファイルで定義されたデータを永続化するために：
Now, to persist the data defined in your fixture file, just run the following command:

.. code-block:: bash

   $ php app/console doctrine:fixtures:load

フィクスチャファイルでは、トークンは、テストを簡略化するためにハードコードされていますが、実際のユーザーがアカウントに適用されたときに、トークンは、私たちのアフィリエイトクラスでそれをする関数を作成してみましょう、生成する必要があります。お使いのORMファイル内、lifecycleCallbacksセクションにsetTokenValueメソッドを追加することで起動します。
In the fixture file, the tokens are hardcoded to simplify the testing, but when an actual user applies for an account, the token will need to be generated Let’s create a function to do that in our Affiliate class. Start by adding the setTokenValue method to lifecycleCallbacks section, inside your ORM file:

src//Ibw/JobeetBundle/Resources/config/doctrine/Affiliate.orm.yml

.. code-block:: yaml

   # ...
       lifecycleCallbacks:
           prePersist: [ setCreatedAtValue, setTokenValue ]

次のコマンドを実行する時期を今、setTokenValue法は実体ファイルの内部で生成されます。
Now, the setTokenValue method will be generated inside the entity file when you will run the following command:

.. code-block:: bash

   $ php app/console doctrine:generate:entities IbwJobeetBundle

それでは、メソッドを変更してみましょう：
Let’s modify the method now:

src/Ibw/JobeetBundle/Entity/Affiliate.php

.. code-block:: php

   public function setTokenValue()
   {
     if(!$this->getToken()) {
         $token = sha1($this->getEmail().rand(11111, 99999));
         $this->token = $token;
     }

     return $this;
   }

データをリロードします。
Reload the data:

.. code-block:: bash

   $ php app/console doctrine:fixtures:load

The Job Web Service
-------------------

新しいリソースを作成するときにいつものように、それは最初にルートを定義するのは良いhabbitです：
As always, when you create a new resource, it’s a good habbit to define the route first:

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   IbwJobeetBundle_api:
       pattern: /api/{token}/jobs.{_format}
       defaults: {_controller: "IbwJobeetBundle:Api:list"}
       requirements:
           _format: xml|json|yaml

あなたは、ルーティングファイルを変更した後、通常のように、あなたはキャッシュをクリアする必要があります。
As usually, after you modify a routing file, you need to clear the cache:

.. code-block:: bash

   $ php app/console cache:clear --env=dev
   $ php app/console cache:clear --env=prod

次のステップは、同じアクションを共有するAPIの動作とテンプレートを作成することです。私たちは今ApiControllerという新しいコントローラファイルを作成してみましょう：
The next step is to create the api action and the templates, that will share the same action. Let us now create a new controller file, called ApiController:

src/Ibw/JobeetBundle/Controller/ApiController.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Controller;

   use Symfony\Bundle\FrameworkBundle\Controller\Controller;
   use Symfony\Component\HttpFoundation\Request;
   use Symfony\Component\HttpFoundation\Response;
   use Ibw\JobeetBundle\Entity\Affiliate;
   use Ibw\JobeetBundle\Entity\Job;
   use Ibw\JobeetBundle\Repository\AffiliateRepository;

   class ApiController extends Controller
   {
       public function listAction(Request $request, $token)
       {
           $em = $this->getDoctrine()->getManager();

           $jobs = array();

           $rep = $em->getRepository('IbwJobeetBundle:Affiliate');
           $affiliate = $rep->getForToken($token);

           if(!$affiliate) {
               throw $this->createNotFoundException('This affiliate account does not exist!');
           }

           $rep = $em->getRepository('IbwJobeetBundle:Job');
           $active_jobs = $rep->getActiveJobs(null, null, null, $affiliate->getId());

           foreach ($active_jobs as $job) {
               $jobs[$this->get('router')->generate('ibw_job_show', array('company' => $job->getCompanySlug(), 'location' => $job->getLocationSlug(), 'id' => $job->getId(), 'position' => $job->getPositionSlug()), true)] = $job->asArray($request->getHost());
           }

           $format = $request->getRequestFormat();
           $jsonData = json_encode($jobs);

           if ($format == "json") {
               $headers = array('Content-Type' => 'application/json');
               $response = new Response($jsonData, 200, $headers);

               return $response;
           }

           return $this->render('IbwJobeetBundle:Api:jobs.' . $format . '.twig', array('jobs' => $jobs));
       }
   }

彼のトークンを使用してアフィリエイトを取得するには、私たちはgetForToken（）メソッドを作成します。アフィリエイトアカウントが起動された場合、この方法はまた、検証ので、これをもう一度チェックする私達の必要はない。今まで、私たちはまだAffiliateRepositoryを使用していないので、それは存在しません。それを作成するには、その後、次のようなORMファイルを変更するには、エンティティを生成する前に、使用されたコマンドを実行します。
To retrieve the affiliate using his token, we will create the getForToken() method. This method also verifies if the affiliate account is activated, so there is no need for us to check this one more time. Until now, we haven’t used the AffiliateRepository yet, so it doesn’t exist. To create it, modify the ORM file as following, then run the command you used before to generate the entities.

src/Ibw/JobeetBundle/Resources/config/doctrine/Affiliate.orm.yml

.. code-block:: yaml

   Ibw\JobeetBundle\Entity\Affiliate:
       type: entity
       repositoryClass: Ibw\JobeetBundle\Repository\AffiliateRepository
       # ...

作成した後は、使用する準備ができている。
Once created, it is ready to be used:

src/Ibw/JobeetBundle/Repository/AffiliateRepository.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Repository;

   use Doctrine\ORM\EntityRepository;

   /**
    * AffiliateRepository
    *
    * This class was generated by the Doctrine ORM. Add your own custom
    * repository methods below.
    */
   class AffiliateRepository extends EntityRepository
   {
       public function getForToken($token)
       {
           $qb = $this->createQueryBuilder('a')
               ->where('a.is_active = :active')
               ->setParameter('active', 1)
               ->andWhere('a.token = :token')
               ->setParameter('token', $token)
               ->setMaxResults(1)
           ;

           try{
               $affiliate = $qb->getQuery()->getSingleResult();
           } catch(\Doctrine\Orm\NoResultException $e){
               $affiliate = null;
           }

           return $affiliate;
       }
   }

彼のトークンによってアフィリエイトを識別した後、私たちは選択したカテゴリに属する、アフィリエイトに彼が必要な仕事を与えるためにgetActiveJobs（）メソッドを使用します。あなたが今、あなたのJobRepositoryファイルを開くと、getActiveJobs（）メソッドは、関連会社との任意の接続を共有していないことがわかります。私たちは、そのメソッドを再利用したいので、私たちはその中のいくつかの変更を行う必要があります。
After identifying the affiliate by his token, we will use the getActiveJobs() method to give the affiliate the jobs he required, belonging to the selected categories. If you open your JobRepository file now, you will see that the getActiveJobs() method doesn’t share any connection with the affiliates. Because we want to reuse that method, we need to make some modifications inside of it:

src/Ibw/JobeetBundle/Repository/JobRepository.php

.. code-block:: php

   // ...

   public function getActiveJobs($category_id = null, $max = null, $offset = null, $affiliate_id = null)
   {
     $qb = $this->createQueryBuilder('j')
         ->where('j.expires_at > :date')
         ->setParameter('date', date('Y-m-d H:i:s', time()))
         ->andWhere('j.is_activated = :activated')
         ->setParameter('activated', 1)
         ->orderBy('j.expires_at', 'DESC');

     if($max) {
         $qb->setMaxResults($max);
     }

     if($offset) {
         $qb->setFirstResult($offset);
     }

     if($category_id) {
         $qb->andWhere('j.category = :category_id')
             ->setParameter('category_id', $category_id);
     }
     // j.category c, c.affiliate a
     if($affiliate_id) {
         $qb->leftJoin('j.category', 'c')
            ->leftJoin('c.affiliates', 'a')
            ->andWhere('a.id = :affiliate_id')
            ->setParameter('affiliate_id', $affiliate_id)
         ;
     }

     $query = $qb->getQuery();

     return $query->getResult();
   }

   // ...

ご覧のように、私たちはasArrayという関数を（）を使用して、ジョブ配列を移入します。のは、それを定義してみましょう：
As you can see, we populate the jobs array using a function called asArray(). Let’s define it:

src/Ibw/JobeetBundle/Entity/Job.php

.. code-block:: php

   public function asArray($host)
   {
       return array(
           'category'     => $this->getCategory()->getName(),
           'type'         => $this->getType(),
           'company'      => $this->getCompany(),
           'logo'         => $this->getLogo() ? 'http://' . $host . '/uploads/jobs/' . $this->getLogo() : null,
           'url'          => $this->getUrl(),
           'position'     => $this->getPosition(),
           'location'     => $this->getLocation(),
           'description'  => $this->getDescription(),
           'how_to_apply' => $this->getHowToApply(),
           'expires_at'   => $this->getCreatedAt()->format('Y-m-d H:i:s'),
       );
   }

The xml Format
---------------

xml形式をサポートするテンプレートを作成するのと同じくらい簡単です。
Supporting the xml format is as simple as creating a template:

src/Ibw/JobeetBundle/Resources/views/Api/jobs.xml.twig

.. code-block:: jinja

   <?xml version="1.0" encoding="utf-8"?>
   <jobs>
   {% for url, job in jobs %}
       <job url="{{ url }}">
   {% for key,value in job %}
           <{{ key }}>{{ value }}</{{ key }}>
   {% endfor %}
       </job>
   {% endfor %}
   </jobs>

The json Format
---------------

JSON形式が似てサポートします。
Support the JSON format is similar:

src/Ibw/JobeetBundle/Resources/views/Api/jobs.json.twig

.. code-block:: jinja

   {% for url, job in jobs %}
   {% i = 0, count(jobs), ++i %}
   [
       "url":"{{ url }}",
   {% for key, value in job %} {% j = 0, count(key), ++j %}
       "{{ key }}":"{% if j == count(key)%} {{ json_encode(value) }}, {% else %} {{ json_encode(value) }}
                    {% endif %}"
   {% endfor %}]
   {% endfor %}

The yaml Format
---------------

src/Ibw/JobeetBundle/Resources/views/Api/jobs.yaml.twig

.. code-block:: jinja

   {% for url,job in jobs %}
       Url: {{ url }}
   {% for key, value in job %}
           {{ key }}: {{ value }}
   {% endfor %}
   {% endfor %}

あなたは、非有効なトークンを使用してWebサービスを呼び出そうとした場合は、すべての形式の応答として404ページを受け取ります。ます。http：//jobeet.local/app_dev.php/api/sensio-labs/jobs.xmlまたはhttp：//jobeet.local/app_dev.php/api/symfony今まであなた熟達は、以下のリンクにアクセスするかを確認するために、 /jobs.xml。お好みの形式に応じて、URLに拡張子を変更します。
If you try to call the web service with a non-valid token, you will receive a 404 page as a response, for all the formats. To see what you accomplished until now, access the following links: http://jobeet.local/app_dev.php/api/sensio-labs/jobs.xml or http://jobeet.local/app_dev.php/api/symfony/jobs.xml. Change the extension in the URL, depending on which format you prefer.

Web Service Tests
-----------------

src/Ibw/JobeetBundle/Tests/Controller/ApiControllerTest.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Tests\Controller;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
   use Symfony\Component\DomCrawler\Crawler;
   use Symfony\Component\HttpFoundation\HttpExceptionInterface;

   class ApiControllerTest extends WebTestCase
   {
       private $em;

       private $application;

       public function setUp()
       {
           static::$kernel = static::createKernel();
           static::$kernel->boot();

           $this->application = new Application(static::$kernel);

           // drop the database
           $command = new DropDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:drop',
               '--force' => true
           ));
           $command->run($input, new NullOutput());

           // we have to close the connection after dropping the database so we don't get "No database selected" error
           $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
           if ($connection->isConnected()) {
               $connection->close();
           }

           // create the database
           $command = new CreateDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:create',
           ));
           $command->run($input, new NullOutput());

           // create schema
           $command = new CreateSchemaDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:schema:create',
           ));
           $command->run($input, new NullOutput());

           // get the Entity Manager
           $this->em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           // load fixtures
           $client = static::createClient();
           $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
           $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
           $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
           $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
           $executor->execute($loader->getFixtures());
       }

       public function testList()
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/api/sensio-labs/jobs.xml');

           $this->assertEquals('Ibw\JobeetBundle\Controller\ApiController::listAction', $client->getRequest()->attributes->get('_controller'));
           $this->assertTrue($crawler->filter('description')->count() == 32);

           $crawler = $client->request('GET', '/api/sensio-labs87/jobs.xml');

           $this->assertTrue(404 === $client->getResponse()->getStatusCode());

           $crawler = $client->request('GET', '/api/symfony/jobs.xml');

           $this->assertTrue(404 === $client->getResponse()->getStatusCode());

           $crawler = $client->request('GET', '/api/sensio-labs/jobs.json');

           $this->assertEquals('Ibw\JobeetBundle\Controller\ApiController::listAction', $client->getRequest()->attributes->get('_controller'));
           $this->assertRegExp('/"category"\:"Programming"/', $client->getResponse()->getContent());

           $crawler = $client->request('GET', '/api/sensio-labs87/jobs.json');

           $this->assertTrue(404 === $client->getResponse()->getStatusCode());

           $crawler = $client->request('GET', '/api/sensio-labs/jobs.yaml');
           $this->assertRegExp('/category\: Programming/', $client->getResponse()->getContent());

           $this->assertEquals('Ibw\JobeetBundle\Controller\ApiController::listAction', $client->getRequest()->attributes->get('_controller'));

           $crawler = $client->request('GET', '/api/sensio-labs87/jobs.yaml');

           $this->assertTrue(404 === $client->getResponse()->getStatusCode());
       }
   }

ApiControllerTestファイル内では、要求の形式が正しく受信され、要求されたページが正しく返されることをテストします。
Inside the ApiControllerTest file, we test that the request formats are correctly received and the pages requested are correctly returned.

The Affiliate Application Form
------------------------------

今、Webサービスが使用できる状態であること、のアフィリエイト用のアカウント作成フォームを作成してみましょう。そのためには、各フィールドの検証ルールを、HTMLフォームを書き実装する必要があるデータベース、表示エラー·メッセージに格納し、エラーの場合はフィールドを再設定するための値を処理します。 
まず、AffiliateControllerという名前の新しいコントローラファイルを、作成します。
Now that the web service is ready to be used, let’s create the account creation form for affiliates. For that, you need to write the HTML form, implement validation rules for each field, process the values to store them in a database, display error messages and repopulate fields in case of errors.
First, create a new controller file, named AffiliateController:

src/Ibw/JobeetBundle/Controller/AffiliateController.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Controller;

   use Symfony\Bundle\FrameworkBundle\Controller\Controller;
   use Ibw\JobeetBundle\Entity\Affiliate;
   use Ibw\JobeetBundle\Form\AffiliateType;
   use Symfony\Component\HttpFoundation\Request;
   use Ibw\JobeetBundle\Entity\Category;

   class AffiliateController extends Controller
   {
       // Your code goes here
   }

その後、レイアウト内のアフィリエイトリンクを変更します。
Then, change the Affiliates link in the layout:

src/Ibw/JobeetBundle/Resources/views/layout.html.twig

.. code-block:: html+jinja

   <!-- ... -->
       <li class="last"><a href="{{ path('ibw_affiliate_new') }}">Become an affiliate</a></li>
   <!-- ... -->

今、私たちはあなただけ先にそれを修正されたリンクからのルートに合わせてアクションを作成する必要があります。
Now, we need to create an action to match the route from the link you just modified it earlier:

src/Ibw/JobeetBundle/Controller/AffiliateController.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Controller;

   use Symfony\Bundle\FrameworkBundle\Controller\Controller;
   use Ibw\JobeetBundle\Entity\Affiliate;
   use Ibw\JobeetBundle\Form\AffiliateType;
   use Symfony\Component\HttpFoundation\Request;
   use Ibw\JobeetBundle\Entity\Category;

   class AffiliateController extends Controller
   {
       public function newAction()
       {
           $entity = new Affiliate();
           $form = $this->createForm(new AffiliateType(), $entity);

           return $this->render('IbwJobeetBundle:Affiliate:affiliate_new.html.twig', array(
               'entity' => $entity,
               'form'   => $form->createView(),
           ));
       }
   }

私たちは、ルートの名前を持って、私たちは行動を持っていますが、私たちはルートを持っていません。それでは、それを作成してみましょう：
We have the name of the route, we have the action, but we do not have the route. so let’s create it:

src/Ibw/JobeetBundle/Resources/config/routing/affiliate.yml

.. code-block:: yaml

   ibw_affiliate_new:
       pattern:  /new
       defaults: { _controller: "IbwJobeetBundle:Affiliate:new" }

また、ルーティングファイルにこれを追加します。
Also, add this to your routing file:

src/Ibw/JobeetBundle/Resources/config/routing.yml

.. code-block:: yaml

   # ...

   IbwJobeetBundle_ibw_affiliate:
       resource: "@IbwJobeetBundle/Resources/config/routing/affiliate.yml"
       prefix:   /affiliate

フォームファイルも作成する必要がある。そのうちのいくつかは、エンドユーザーが編集であってはなりませんので、しかし、アフィリエイトは複数のフィールドがあっても、私たちは、それらをすべて表示しません。あなたのアフィリエイトのフォームを作成します。
The form file also needs to be created. But, even if the Affiliate has more fields, we won’t display them all, because some of them must not be editable by the end user. Create your Affiliate form:

src/Ibw/JobeetBundle/Form/AffiliateType.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Form;

   use Symfony\Component\Form\AbstractType;
   use Symfony\Component\Form\FormBuilderInterface;
   use Symfony\Component\OptionsResolver\OptionsResolverInterface;
   use Ibw\JobeetBundle\Entity\Affiliate;
   use Ibw\JobeetBundle\Entity\Category;

   class AffiliateType extends AbstractType
   {
       public function buildForm(FormBuilderInterface $builder, array $options)
       {
           $builder
               ->add('url')
               ->add('email')
               ->add('categories', null, array('expanded'=>true))
           ;
       }

       public function setDefaultOptions(OptionsResolverInterface $resolver)
       {
           $resolver->setDefaults(array(
               'data_class' => 'Ibw\JobeetBundle\Entity\Affiliate',
           ));
       }

       public function getName()
       {
           return 'affiliate';
       }
   }

今、私たちは、フォームがそれに提出されたデータを適用した後にアフィリエイトオブジェクトが有効であるかどうかを決定する必要があります。これを行うには、バリデーションファイルに次のコードを追加します。
Now, we need to decide whether or not the Affiliate object is valid after the form has applied the submitted data to it. To do this, add the following code to your validation file:

src/Ibw/JobeetBundle/Resources/config/validation.yml

.. code-block:: yaml

   # ...

   Ibw\JobeetBundle\Entity\Affiliate:
       constraints:
           - Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity: email
       properties:
           url:
               - Url: ~
           email:
               - NotBlank: ~
               - Email: ~

検証スキーマでは、UniqueEntityという新しいバリデータを使用しました。これはDoctrineのエンティティ内の特定のフィールド（またはフィールド）がユニークな（ある）であることを検証します。これは、一般的にシステムに既に存在する電子メールアドレスを使用して登録する新しいユーザーを防止するために、例えば、使用される。 
検証制約を適用した後にキャッシュをクリアすることを忘れないでください！ 
最後に、フォームのビューをあまりにも作成してみましょう：
In the validation schema, we used a new validator, called UniqueEntity.  It validates that a particular field (or fields) in a Doctrine entity is (are) unique. This is commonly used, for example, to prevent a new user to register using an email address that already exists in the system.
Don’t forget to clear your cache after applying the validation constraints!
Finally, let’s create the view for the form too:

src/Ibw/JobeetBundle/Resources/views/Affiliate/affiliate_new.html.twig

.. code-block:: html+jinja

   {% extends 'IbwJobeetBundle::layout.html.twig' %}

   {% set form_themes = _self %}

   {% block form_errors %}
   {% spaceless %}
       {% if errors|length > 0 %}
           <ul class="error_list">
               {% for error in errors %}
                   <li>{{ error.messageTemplate|trans(error.messageParameters, 'validators') }}</li>
               {% endfor %}
           </ul>
       {% endif %}
   {% endspaceless %}
   {% endblock form_errors %}

   {% block stylesheets %}
       {{ parent() }}
       <link rel="stylesheet" href="{{ asset('bundles/ibwjobeet/css/job.css') }}" type="text/css" media="all" />
   {% endblock %}

   {% block content %}
       <h1>Become an affiliate</h1>
           <form action="{{ path('ibw_affiliate_create') }}" method="post" {{ form_enctype(form) }}>
               <table id="job_form">
                   <tfoot>
                       <tr>
                           <td colspan="2">
                               <input type="submit" value="Submit" />
                           </td>
                       </tr>
                   </tfoot>
                   <tbody>
                       <tr>
                           <th>{{ form_label(form.url) }}</th>
                           <td>
                               {{ form_errors(form.url) }}
                               {{ form_widget(form.url) }}
                           </td>
                       </tr>
                       <tr>
                           <th>{{ form_label(form.email) }}</th>
                           <td>
                               {{ form_errors(form.email) }}
                               {{ form_widget(form.email) }}
                           </td>
                       </tr>
                       <tr>
                           <th>{{ form_label(form.categories) }}</th>
                           <td>
                               {{ form_errors(form.categories) }}
                               {{ form_widget(form.categories) }}
                           </td>
                       </tr>
                   </tbody>
               </table>
           {{ form_end(form) }}
   {% endblock %}

ユーザーがフォームを送信すると、有効な場合は、フォームデータは、データベースに永続化されなければならない。あなたのアフィリエイトコントローラに新しいcreateアクションを追加します。
When the user submits a form, the form data must be persisted into database, if valid. Add the new create action to your Affiliate controller:

.. code-block:: php

   src/Ibw/JobeetBundle/Controller/AffiliateController.php

   class AffiliateController extends Controller
   {
       // ...

       public function createAction(Request $request)
       {
           $affiliate = new Affiliate();
           $form = $this->createForm(new AffiliateType(), $affiliate);
           $form->bind($request);
           $em = $this->getDoctrine()->getManager();

           if ($form->isValid()) {

               $formData = $request->get('affiliate');
               $affiliate->setUrl($formData['url']);
               $affiliate->setEmail($formData['email']);
               $affiliate->setIsActive(false);

               $em->persist($affiliate);
               $em->flush();

               return $this->redirect($this->generateUrl('ibw_affiliate_wait'));
           }

           return $this->render('IbwJobeetBundle:Affiliate:affiliate_new.html.twig', array(
               'entity' => $affiliate,
               'form'   => $form->createView(),
           ));
       }
   }

提出される場合は、createアクションが実行ので、ルートを定義する必要がありますされています。
When submitting, the create action is performed, so we need to define the route:

src/Ibw/JobeetBundle/Resources/config/routing/affiliate.yml

.. code-block:: yaml

   # ...

   ibw_affiliate_create:
       pattern: /create
       defaults: { _controller: "IbwJobeetBundle:Affiliate:create" }
       requirements: { _method: post }

アフィリエイト登録された、彼は待ちページにリダイレクトされます。それでは、そのアクションを定義し、あまりにもビューを作成してみましょう：
After the affiliate registers, he is redirected to a waiting page. Let’s define that action and create the view too:

   src/Ibw/JobeetBundle/Controller/AffiliateController.php

.. code-block:: php

   class AffiliateController extends Controller
   {
       // ...

       public function waitAction()
       {
           return $this->render('IbwJobeetBundle:Affiliate:wait.html.twig');
       }
   }

src/Ibw/JobeetBundle/Resources/views/Affiliate/wait.html.twig

.. code-block:: html+jinja

   {% extends "IbwJobeetBundle::layout.html.twig" %}

   {% block content %}
       <div class="content">
           <h1>Your affiliate account has been created</h1>
           <div style="padding: 20px">
               Thank you!
               You will receive an email with your affiliate token
               as soon as your account will be activated.
           </div>
       </div>
   {% endblock %}

さて、ルート：
Now, the route:

src/Ibw/JobeetBundle/Resources/config/routing/affiliate.yml

.. code-block:: yaml

   # ...

   ibw_affiliate_wait:
       pattern: /wait
       defaults: { _controller: "IbwJobeetBundle:Affiliate:wait" }

動作するために、ルートに定義したら、キャッシュをクリアする必要があります。 
あなたがホームページでアフィリエイトリンクをクリックした場合さて、あなたはアフィリエイトのフォームページに転送されます。
After defining to routes, in order to work, you need to clear the cache.
Now, if you click on the Affiliates link on the homepage, you will be directed to the affiliate form page.

Tests
-----

最後のステップは新しい機能のためのいくつかの機能テストを書くことです。
The last step is to write some functional tests for the new feature.

src/Ibw/JobeetBundle/Tests/Controller/AffiliateControllerTest.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Tests\Controller;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;
   use Symfony\Component\DomCrawler\Crawler;

   class AffiliateControllerTest extends WebTestCase
   {
       private $em;
       private $application;

       public function setUp()
       {
           static::$kernel = static::createKernel();
           static::$kernel->boot();

           $this->application = new Application(static::$kernel);

           // drop the database
           $command = new DropDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:drop',
               '--force' => true
           ));
           $command->run($input, new NullOutput());

           // we have to close the connection after dropping the database so we don't get "No database selected" error
           $connection = $this->application->getKernel()->getContainer()->get('doctrine')->getConnection();
           if ($connection->isConnected()) {
               $connection->close();
           }

           // create the database
           $command = new CreateDatabaseDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:database:create',
           ));
           $command->run($input, new NullOutput());

           // create schema
           $command = new CreateSchemaDoctrineCommand();
           $this->application->add($command);
           $input = new ArrayInput(array(
               'command' => 'doctrine:schema:create',
           ));
           $command->run($input, new NullOutput());

           // get the Entity Manager
           $this->em = static::$kernel->getContainer()
               ->get('doctrine')
               ->getManager();

           // load fixtures
           $client = static::createClient();
           $loader = new \Symfony\Bridge\Doctrine\DataFixtures\ContainerAwareLoader($client->getContainer());
           $loader->loadFromDirectory(static::$kernel->locateResource('@IbwJobeetBundle/DataFixtures/ORM'));
           $purger = new \Doctrine\Common\DataFixtures\Purger\ORMPurger($this->em);
           $executor = new \Doctrine\Common\DataFixtures\Executor\ORMExecutor($this->em, $purger);
           $executor->execute($loader->getFixtures());
       }

       public function testAffiliateForm()
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/affiliate/new');

           $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::newAction', $client->getRequest()->attributes->get('_controller'));

           $form = $crawler->selectButton('Submit')->form(array(
               'affiliate[url]' => 'http://sensio-labs.com/',
               'affiliate[email]' => 'jobeet@example.com'
           ));

           $client->submit($form);
           $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::createAction', $client->getRequest()->attributes->get('_controller'));

           $kernel = static::createKernel();
           $kernel->boot();
           $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

           $query = $em->createQuery('SELECT count(a.email) FROM IbwJobeetBundle:Affiliate a WHERE a.email = :email');
           $query->setParameter('email', 'jobeet@example.com');
           $this->assertEquals(1, $query->getSingleScalarResult());

           $crawler = $client->request('GET', '/affiliate/new');
           $form = $crawler->selectButton('Submit')->form(array(
               'affiliate[email]'        => 'not.an.email',
           ));
           $crawler = $client->submit($form);

           // check if we have 1 errors
           $this->assertTrue($crawler->filter('.error_list')->count() == 1);
           // check if we have error on affiliate_email field
           $this->assertTrue($crawler->filter('#affiliate_email')->siblings()->first()->filter('.error_list')->count() == 1);
       }

       public function testCreate()
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/affiliate/new');
           $form = $crawler->selectButton('Submit')->form(array(
               'affiliate[url]' => 'http://sensio-labs.com/',
               'affiliate[email]' => 'address@example.com'
           ));

           $client->submit($form);
           $client->followRedirect();

           $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::waitAction', $client->getRequest()->attributes->get('_controller'));

           return $client;
       }

       public function testWait()
       {
           $client = static::createClient();
           $crawler = $client->request('GET', '/affiliate/wait');

           $this->assertEquals('Ibw\JobeetBundle\Controller\AffiliateController::waitAction', $client->getRequest()->attributes->get('_controller'));
       }
   }

The Affiliate Backend
---------------------

バックエンドのために、私たちはSonataAdminBundleで動作します。私たちは前にも言ったように、アフィリエイトのレジスタの後、彼は自分のアカウントをアクティブにするために管理者を待つ必要がある。だから、管理者は関連会社のページにアクセスしますと、彼は彼の生産性を助けるために、唯一の不活性化されたアカウントが表示されます。 
まず第一に、あなたは、あなたのservices.ymlファイル内の新しいアフィリエイトサービスを宣言する必要があります。
For the backend, we will work with SonataAdminBundle. As we said before, after an affiliate registers, he needs to wait for the admin to activate his account. So, when the admin will access the affiliates page, he will see only the inactivated accounts, to help him be more productive.
First of all, you need to declare the new affiliate service inside your services.yml file:

src/Ibw/JobeetBundle/Resources/config/services.yml

.. code-block:: yaml

   # ...
   ibw.jobeet.admin.affiliate:
     class: Ibw\JobeetBundle\Admin\AffiliateAdmin
     tags:
         - { name: sonata.admin, manager_type: orm, group: jobeet, label: Affiliates }
     arguments:
         - ~
         - Ibw\JobeetBundle\Entity\Affiliate
         - 'IbwJobeetBundle:AffiliateAdmin'

その後、管理ファイルを作成します。
After that, create the Admin file:

.. code-block:: php

   src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php

   namespace Ibw\JobeetBundle\Admin;

   use Sonata\AdminBundle\Admin\Admin;
   use Sonata\AdminBundle\Datagrid\ListMapper;
   use Sonata\AdminBundle\Datagrid\DatagridMapper;
   use Sonata\AdminBundle\Validator\ErrorElement;
   use Sonata\AdminBundle\Form\FormMapper;
   use Sonata\AdminBundle\Show\ShowMapper;
   use Ibw\JobeetBundle\Entity\Affiliate;

   class AffiliateAdmin extends Admin
   {
       protected $datagridValues = array(
           '_sort_order' => 'ASC',
           '_sort_by' => 'is_active'
       );

       protected function configureFormFields(FormMapper $formMapper)
       {
           $formMapper
               ->add('email')
               ->add('url')
           ;
       }

       protected function configureDatagridFilters(DatagridMapper $datagridMapper)
       {
           $datagridMapper
               ->add('email')
               ->add('is_active');
       }

       protected function configureListFields(ListMapper $listMapper)
       {
           $listMapper
               ->add('is_active')
               ->addIdentifier('email')
               ->add('url')
               ->add('created_at')
               ->add('token')
           ;
       }
   }

管理者を助けるために、私たちは、不活性化されたアカウントを表示したい。これをfalseに'is_active'フィルタを設定することで行うことができます。
To help the administrator, we want to display only the inactivated accounts. This can be made by setting the ‘is_active’ filter to false:

src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php

.. code-block:: php

   // ...
   protected $datagridValues = array(
     '_sort_order' => 'ASC',
     '_sort_by' => 'is_active',
     'is_active' => array('value' => 2) // The value 2 represents that the displayed affiliate accounts are not activated yet
   );

   // ...

さて、AffiliateAdminコントローラファイルを作成します。
Now, create the AffiliateAdmin controller file:

src/Ibw/JobeetBundle/Controller/AffiliateAdminController.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Controller;

   use Sonata\AdminBundle\Controller\CRUDController as Controller;
   use Sonata\DoctrineORMAdminBundle\Datagrid\ProxyQuery as ProxyQueryInterface;
   use Symfony\Component\HttpFoundation\RedirectResponse;

   class AffiliateAdminController extends Controller
   {
       // Your code goes here
   }

それではアクティベートを作成して、バッチアクションを非アクティブにしてみましょう：
Let’s create the activate and deactivate batch actions:

src/Ibw/JobeetBundle/Controller/AffiliateAdminController.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Controller;

   use Sonata\AdminBundle\Controller\CRUDController as Controller;
   use Sonata\DoctrineORMAdminBundle\Datagrid\ProxyQuery as ProxyQueryInterface;
   use Symfony\Component\HttpFoundation\RedirectResponse;

   class AffiliateAdminController extends Controller
   {
       public function batchActionActivate(ProxyQueryInterface $selectedModelQuery)
       {
           if($this->admin->isGranted('EDIT') === false || $this->admin->isGranted('DELETE') === false) {
               throw new AccessDeniedException();
           }

           $request = $this->get('request');
           $modelManager = $this->admin->getModelManager();

           $selectedModels = $selectedModelQuery->execute();

           try {
               foreach($selectedModels as $selectedModel) {
                   $selectedModel->activate();
                   $modelManager->update($selectedModel);
               }
           } catch(\Exception $e) {
               $this->get('session')->getFlashBag()->add('sonata_flash_error', $e->getMessage());

               return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
           }

           $this->get('session')->getFlashBag()->add('sonata_flash_success',  sprintf('The selected accounts have been activated'));

           return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
       }

       public function batchActionDeactivate(ProxyQueryInterface $selectedModelQuery)
       {
           if($this->admin->isGranted('EDIT') === false || $this->admin->isGranted('DELETE') === false) {
               throw new AccessDeniedException();
           }

           $request = $this->get('request');
           $modelManager = $this->admin->getModelManager();

           $selectedModels = $selectedModelQuery->execute();

           try {
               foreach($selectedModels as $selectedModel) {
                   $selectedModel->deactivate();
                   $modelManager->update($selectedModel);
               }
           } catch(\Exception $e) {
               $this->get('session')->getFlashBag()->add('sonata_flash_error', $e->getMessage());

               return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
           }

           $this->get('session')->getFlashBag()->add('sonata_flash_success',  sprintf('The selected accounts have been deactivated'));

           return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
       }
   }

新しいバッチアクションを機能させるには、私たちは、管理クラスからgetBatchActionsでそれらを追加する必要があります。
For the new batch actions to be functional, we have to add them in the getBatchActions from the Admin class:

src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php

.. code-block:: php

   class AffiliateAdmin extends Admin
   {
       // ...

       public function getBatchActions()
       {
           $actions = parent::getBatchActions();

           if($this->hasRoute('edit') && $this->isGranted('EDIT') && $this->hasRoute('delete') && $this->isGranted('DELETE')) {
               $actions['activate'] = array(
                   'label'            => 'Activate',
                   'ask_confirmation' => true
               );

               $actions['deactivate'] = array(
                   'label'            => 'Deactivate',
                   'ask_confirmation' => true
               );
           }

           return $actions;
       }
   }

これが機能するためには、実体ファイルには、2つのメソッドを追加アクティブまたは非アクティブにする必要があります。
For this to work, you need to add the two methods, activate and deactivate, in the entity file:

src/Ibw/JobeetBundle/Entity/Affiliate.php

.. code-block:: php

   // ...

   public function activate()
   {
     if(!$this->getIsActive()) {
         $this->setIsActive(true);
     }

     return $this->is_active;
   }

   public function deactivate()
   {
     if($this->getIsActive()) {
         $this->setIsActive(false);
     }

     return $this->is_active;
   }

それでは、二つの個別のアクションを作成し、各項目ごとに、アクティブまたは非アクティブにしてみましょう。第一に、私たちは彼らのためにルートを作成します。それは、あなたの管理クラスで、あなたがconfigureRoutes機能を拡張します理由です。
Let’s now create two individual actions, activate and deactivate, for each item. Firstly, we will create routes for them. That’s why, in your Admin class, you will extend the configureRoutes function:

src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php

.. code-block:: php

   use Sonata\AdminBundle\Route\RouteCollection;

   class AffiliateAdmin extends Admin
   {
       // ...

       protected function configureRoutes(RouteCollection $collection) {
           parent::configureRoutes($collection);

           $collection->add('activate',
               $this->getRouterIdParameter().'/activate')
           ;

           $collection->add('deactivate',
               $this->getRouterIdParameter().'/deactivate')
           ;
       }
   }

それはAdminControllerでアクションを実装してみましょう：
It’s time to implement the actions in the AdminController:

src/Ibbw/JobeetBundle/Controller/AffiliateAdminController.php

.. code-block:: php

   class AffiliateAdminController extends Controller
   {
       // ...

       public function activateAction($id)
       {
           if($this->admin->isGranted('EDIT') === false) {
               throw new AccessDeniedException();
           }

           $em = $this->getDoctrine()->getManager();
           $affiliate = $em->getRepository('IbwJobeetBundle:Affiliate')->findOneById($id);

           try {
               $affiliate->setIsActive(true);
               $em->flush();
           } catch(\Exception $e) {
               $this->get('session')->getFlashBag()->add('sonata_flash_error', $e->getMessage());

               return new RedirectResponse($this->admin->generateUrl('list', $this->admin->getFilterParameters()));
           }

           return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));

       }

       public function deactivateAction($id)
       {
           if($this->admin->isGranted('EDIT') === false) {
               throw new AccessDeniedException();
           }

           $em = $this->getDoctrine()->getManager();
           $affiliate = $em->getRepository('IbwJobeetBundle:Affiliate')->findOneById($id);

           try {
               $affiliate->setIsActive(false);
               $em->flush();
           } catch(\Exception $e) {
               $this->get('session')->getFlashBag()->add('sonata_flash_error', $e->getMessage());

               return new RedirectResponse($this->admin->generateUrl('list', $this->admin->getFilterParameters()));
           }

           return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
       }
   }

今、新たな付加アクションボタン用のテンプレートを作成します。
Now, create the templates for the new added action buttons:

src/Ibw/JobeetBundle/Resources/views/AffiliateAdmin/list__action_activate.html.twig

.. code-block:: html+jinja

   {% if admin.isGranted('EDIT', object) and admin.hasRoute('activate') %}
       <a href="{{ admin.generateObjectUrl('activate', object) }}" class="btn edit_link" title="{{ 'action_activate'|trans({}, 'SonataAdminBundle') }}">
           <i class="icon-edit"></i>
           {{ 'activate'|trans({}, 'SonataAdminBundle') }}
       </a>
   {% endif %}

src/Ibw/JobeetBundle/Resources/views/AffiliateAdmin/list__action_deactivate.html.twig

.. code-block:: html+jinja

   {% if admin.isGranted('EDIT', object) and admin.hasRoute('deactivate') %}
       <a href="{{ admin.generateObjectUrl('deactivate', object) }}" class="btn edit_link" title="{{ 'action_deactivate'|trans({}, 'SonataAdminBundle') }}">
           <i class="icon-edit"></i>
           {{ 'deactivate'|trans({}, 'SonataAdminBundle') }}
       </a>
   {% endif %}

彼らがページに表示されるように、それぞれが個別にアカウントに管理ファイルの内部では、、、configureListFields関数に新しいアクションとボタンを追加します。
Inside your Admin file, add the new actions and buttons to the configureListFields function, so that they would appear on the page, to each account individually:

src/Ibw/JobeetBundle/Admin/AffiliateAdmin.php

.. code-block:: php

   class AffiliateAdmin extends Admin
   {
       // ...

       protected function configureListFields(ListMapper $listMapper)
       {
           $listMapper
               ->add('is_active')
               ->addIdentifier('email')
               ->add('url')
               ->add('created_at')
               ->add('token')
               ->add('_action', 'actions', array( 'actions' => array('activate' => array('template' => 'IbwJobeetBundle:AffiliateAdmin:list__action_activate.html.twig'),
                   'deactivate' => array('template' => 'IbwJobeetBundle:AffiliateAdmin:list__action_deactivate.html.twig'))))
           ;
       }
       /// ...
   }

今、あなたのキャッシュをクリアし、それを試してみてください！ 
つまり、今日のすべてです！明日は、自分のアカウントがアクティブになったときに会社が受け取る電子メールの世話をします。
Now, clear your cache and try it on!
That’s all for today! Tomorrow, we will take care of the emails the affiliates will receive when their accounts have been activated.

.. include:: common/license.rst.inc
