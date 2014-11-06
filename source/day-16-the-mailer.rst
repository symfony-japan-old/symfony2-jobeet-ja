16日目: メール送信
==================

.. include:: common/original.rst.inc

| 昨日は、 Jobeet に読み取り専用の Web サービスを追加しました。
| 現在、アフィリエイトはアカウントの作成はできますが、使用する前に管理者によってアクティブ化される必要があります。
| アフィリエイトがトークンを受け取るためには、メール通知を実装する必要があります。今日の行う作業はそれになります。
| Symfony フレームワークは最高の PHP のメール送信ソリューションの一つ、 Swift Mailer がバンドルされています。
| もちろん、ライブラリは完全に、 Symfony と統合されており、いくつかの素晴らしい機能がデフォルトの機能の上に追加されています。
| では、アカウントがアクティブ化された際アフィリエイトにトークンを通知するため、単純な電子メールを送ることから始めましょう。
| しかし、最初に、ご使用の環境を設定する必要があります。

..
   Yesterday, we added a read-only web service to Jobeet.
   Affiliates can now create an account but it needs to be activated by the administrator before it can be used.
   In order for the affiliate to get its token, we still need to implement the email notification.
   That’s what we will start doing in the coming lines.
   The symfony framework comes bundled with one of the best PHP emailing solution: Swift Mailer.
   Of course, the library is fully integrated with symfony, with some cool features added on top of its default features.
   Let’s start by sending a simple email to notify the affiliate when his account has been activated and to give him the affiliate token.
   But first, you need to configure your environment:

.. code-block:: yaml

   app/config/parameters.yml

   # ...
       # ...
       mailer_transport:  gmail
       mailer_host:       ~
       mailer_user:       address@example.com
       mailer_password:   your_password
       # ...

.. note::

   コードが正しく動作するためには、 address@example.com を現実のメールアドレスに変更し、本当のパスワードを設定する必要があります。
   For the code to work properly, you should change the address@example.com email address to a real one, along with your real password.

| app/config/parameters_test.yml ファイルに同じことを行います。
| 二つのファイルを変更した後、両方のテストおよび開発環境用のキャッシュをクリアします。

..
   Do the same thing in your app/config/parameters_test.yml file.
   After modifying the two files, clear the cache for both test and development environment:

.. code-block:: bash

   $ php app/console cache:clear --env=dev
   $ php app/console cache:clear --env=prod

| メール転送を ``gmail`` に設定したため、 ``mailer_user`` のメールアドレスを置き換える際に、  Google のメールアドレスをで置き換えます。
| メッセージの作成は、メールクライアントでメール作成ボタンをクリックしたときの実行手順と類似していると考えることができます。
| タイトルを設定し、いくつかの受信者を指定し、メッセージを書きます。

..
   Because we set the mailer transport to gmail, when you will replace the email address from “mailer_user”, you will put a google email address.
   You can think of creating a Message as being similar to the steps you perform when you click the compose button in your mail client.
   You give it a subject, specify some recipients and write your message.

メッセージを作成するには、以下となります。

*  Swift_messageのnewInstance() メソッドをコールします。（このオブジェクトの詳細を学ぶためには Swift Mailer の公式ドキュメントを参照してください）
*  setForm() メソッドで送信アドレス( From: )を設定します。
*  setSubject() メソッドでタイトルを設定します。
*  次のいずれかの方法で、受信者を設定します。setTo(), SetCC(), setBcc()。
*  setBodyで本文を設定します。

..
   To create the message, you will:

   *  call the newInstance() methond of Swift_message (refer to the Swift Mailer official documentation to learn more about this object).
   *  set your sender address (From:) with setFrom() method.
   *  set a subject line with setSubject() method.
   *  set recipients with one of these methods: setTo(), setCc() or setBcc().
   *  set a body with setBody().

次のコードで ``activateAction`` を置き換えます。

.. Replace the activate action with the following code:

src/Ibw/JobeetBundle/Controller/AffiliateAdminController.php

.. code-block:: php

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

         $message = \Swift_Message::newInstance()
             ->setSubject('Jobeet affiliate token')
             ->setFrom('address@example.com')
             ->setTo($affiliate->getEmail())
             ->setBody(
                 $this->renderView('IbwJobeetBundle:Affiliate:email.txt.twig', array('affiliate' => $affiliate->getToken())))
         ;

         $this->get('mailer')->send($message);
     } catch(\Exception $e) {
         $this->get('session')->setFlash('sonata_flash_error', $e->getMessage());
     }

     return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
   }

   // ...

| メッセージを送信することは簡単です。メーラーインスタンスで send() メソッドを呼び出し、引数としてメッセージを渡すだけです。
| メッセージ本文のため、 email.txt.twig という新しいファイルを作成し、アフィリエイトの正確な情報を記入します。

..
   Sending the message is then as simple as calling the send() method on the mailer instance and passing the message as an argument.
   For the message body, we created a new file, called email.txt.twig,
   that contains exactly what we want to inform the affiliate about.

src/Ibw/JobeetBundle/Resources/views/Affiliate/email.txt.twig

.. code-block:: jinja

   Your affiliate account has been activated.
   Your secret token is {{affiliate}}.
   You can see the jobs list at the following addresses:
   http://jobeet.local/app_dev.php/api/{{affiliate}}/jobs.xml
   or http://jobeet.local/app_dev.php/api/{{affiliate}}/jobs.json
   or http://jobeet.local/app_dev.php/api/{{affiliate}}/jobs.yaml

| ここで、複数のアフィリエイトアカウントを選択しアクティブ化した場合でも、アクティベーションメールが届くように、
| batchActionActivate にメール機能を追加してみましょう。

..
   Now, let’s add the mailing functionality to the batchActionActivate too,
   so that even if we select multiple affiliate accounts to activate, they will receive their account activation email :

src/Ibw/JobeetBundle/Controller/AffiliateAdminController.php

.. code-block:: php

   // ...

   public function batchActionActivate(ProxyQueryInterface $selectedModelQuery)
   {
     // ...

     try {
         foreach($selectedModels as $selectedModel) {
             $selectedModel->activate();
             $modelManager->update($selectedModel);

             $message = \Swift_Message::newInstance()
                 ->setSubject('Jobeet affiliate token')
                 ->setFrom('address@example.com')
                 ->setTo($selectedModel->getEmail())
                 ->setBody(
                     $this->renderView('IbwJobeetBundle:Affiliate:email.txt.twig', array('affiliate' => $selectedModel->getToken())))
             ;

             $this->get('mailer')->send($message);
         }
     } catch(\Exception $e) {
         $this->get('session')->setFlash('sonata_flash_error', $e->getMessage());

         return new RedirectResponse($this->admin->generateUrl('list',$this->admin->getFilterParameters()));
     }

     // ...
   }

   // ...

テスト
------

| Symfony のメーラーでメール送信する機能を作成しました。次は、作成したものが正しく動くか機能テストを書いて確認しましょう。
| この新しい機能をテストするには、ログインする必要があります。ログインするには、ユーザー名とパスワードが必要になります。
| ユーザー ``admin`` を追加する新しいフィクスチャファイルを作成することから始めます。

..
   Now that we have seen how to send an email with the symfony mailer, let’s write some functional tests to ensure we did the right thing.
   To test this new functionality, we need to be logged in. To log in, we will need an username and a password.
   That’s why we will start by creating a new fixture file, where we add the user admin:

.. code-block:: php

   src/Ibw/JobeetBundle/DataFixtures/ORM/LoadUserData.php

   namespace Ibw\JobeetBundle\DataFixtures\ORM;

   use Doctrine\Common\Persistence\ObjectManager;
   use Doctrine\Common\DataFixtures\AbstractFixture;
   use Doctrine\Common\DataFixtures\FixtureInterface;
   use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
   use Symfony\Component\DependencyInjection\ContainerAwareInterface;
   use Symfony\Component\DependencyInjection\ContainerInterface;
   use Ibw\JobeetBundle\Entity\User;

   class LoadUserData implements FixtureInterface, OrderedFixtureInterface, ContainerAwareInterface
   {
       /**
        * @var ContainerInterface
        */
       private $container;

       /**
        * {@inheritDoc}
        */
       public function setContainer(ContainerInterface $container = null)
       {
           $this->container = $container;
       }

       /**
        * @param \Doctrine\Common\Persistence\ObjectManager $em
        */
       public function load(ObjectManager $em)
       {
           $user = new User();
           $user->setUsername('admin');
           $encoder = $this->container
               ->get('security.encoder_factory')
               ->getEncoder($user)
           ;

           $encodedPassword = $encoder->encodePassword('admin', $user->getSalt());
           $user->setPassword($encodedPassword);

           $em->persist($user);
           $em->flush();
       }

       public function getOrder()
       {
           return 4; // the order in which fixtures will be loaded
       }
   }

| テストにおいて、以前のリクエストで送信したメッセージに関する情報を取得するため、プロファイラ上の SwiftMailer コレクタを使用します。
| では、メールが正しく送信された場合の、いくつかのテストを追加してみましょう。

..
   In the tests, we will use the swiftmailer collector on the profiler to get information about the messages send on the previous requests.
   Now, let’s add some tests to check if the email is sent properly:

src/Ibw/JobeetBundle/Tests/Controller/AffiliateAdminControllerTest.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Tests\Controller;

   use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
   use Symfony\Bundle\FrameworkBundle\Console\Application;
   use Symfony\Component\Console\Output\NullOutput;
   use Symfony\Component\Console\Input\ArrayInput;
   use Doctrine\Bundle\DoctrineBundle\Command\DropDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\CreateDatabaseDoctrineCommand;
   use Doctrine\Bundle\DoctrineBundle\Command\Proxy\CreateSchemaDoctrineCommand;

   class AffiliateAdminControllerTest extends WebTestCase
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

       public function testActivate()
       {
           $client = static::createClient();

           // Enable the profiler for the next request (it does nothing if the profiler is not available)
           $client->enableProfiler();
           $crawler = $client->request('GET', '/login');

           $form = $crawler->selectButton('login')->form(array(
               '_username'      => 'admin',
               '_password'      => 'admin'
           ));

           $crawler = $client->submit($form);
           $crawler = $client->followRedirect();

           $this->assertTrue(200 === $client->getResponse()->getStatusCode());

           $crawler = $client->request('GET', '/admin/ibw/jobeet/affiliate/list');

           $link = $crawler->filter('.btn.edit_link')->link();
           $client->click($link);

           $mailCollector = $client->getProfile()->getCollector('swiftmailer');

           // Check that an e-mail was sent
           $this->assertEquals(1, $mailCollector->getMessageCount());

           $collectedMessages = $mailCollector->getMessages();
           $message = $collectedMessages[0];

           // Asserting e-mail data
           $this->assertInstanceOf('Swift_Message', $message);
           $this->assertEquals('Jobeet affiliate token', $message->getSubject());
           $this->assertRegExp(
               '/Your secret token is symfony/',
               $message->getBody()
           );
       }
   }

| ここでテストを実行する場合、エラーを取得するでしょう。
| 問題を防ぐには、 config_test.yml ファイルに移動し、プロファイラがテスト環境で有効になっていることを確認してください。
| false になっている場合は、 true に変更します。

..
   If you run the test now, you’ll get and error.
   To prevent this for happening, go to your config_test.yml file and make sure that the profiler is enabled in the test environment.
   If it’s set to false, change it to true:

app/config/config_test.yml

.. code-block:: yaml

   # ...

   framework:
       test: ~
       session:
           storage_id: session.storage.mock_file
       profiler:
           enabled: true

   # ...

さて、キャッシュをクリアし、コンソールでテストコマンドを実行し、緑色のバーをお楽しみください。：

.. Now, clear the cache, run the test command in your console and enjoy the green bar :

.. code-block:: bash

   $ phpunit -c app src/Ibw/JobeetBundle/Tests/Controller/AffiliateAdminControllerTest

.. include:: common/license.rst.inc