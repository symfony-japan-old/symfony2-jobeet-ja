Day 11: Testing your Forms
==========================

.. include:: common/original.rst.inc

In day 10, we created our first form with Symfony 2.3. People are now able to post a new job on Jobeet but we ran out of time before we could add some tests. That’s what we will do along these lines.

Submitting a Form
-----------------

Let’s open the JobControllerTest file to add functional tests for the job creation and validation process. At the end of the file, add the following code to get the job creation page:

src.Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   // ...

   public function testJobForm()
   {
       $client = static::createClient();

       $crawler = $client->request('GET', '/job/new');
       $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::newAction', $client->getRequest()->attributes->get('_controller'));
   }

To select forms we will use the selectButton() method. This method can select button tags and submit input tags. Once you have a Crawler representing a button, call the form() method to get a Form instance for the form wrapping the button node:

.. code-block:: php

   $form = $crawler->selectButton('Submit Form')->form();

.. note::

   The above example selects an input of type submit using its value attribute “Submit Form".

When calling the form() method, you can also pass an array of field values that overrides the default ones:

.. code-block:: php

   $form = $crawler->selectButton('submit')->form(array(
       'name' => 'Fabien',
       'my_form[subject]' => 'Symfony Rocks!'
   ));

It is now time to actually select and pass valid values to the form:

src.Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   // ...

   public function testJobForm()
   {
       $client = static::createClient();

       $crawler = $client->request('GET', '/job/new');
       $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::newAction', $client->getRequest()->attributes->get('_controller'));

       $form = $crawler->selectButton('Preview your job')->form(array(
           'job[company]'      => 'Sensio Labs',
           'job[url]'          => 'http://www.sensio.com/',
           'job[file]'         => __DIR__.'/../../../../../web/bundles/ibwjobeet/images/sensio-labs.gif',
           'job[position]'     => 'Developer',
           'job[location]'     => 'Atlanta, USA',
           'job[description]'  => 'You will work with symfony to develop websites for our customers.',
           'job[how_to_apply]' => 'Send me an email',
           'job[email]'        => 'for.a.job@example.com',
           'job[is_public]'    => false,
       ));

       $client->submit($form);
       $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::createAction', $client->getRequest()->attributes->get('_controller'));
   }

The browser also simulates file uploads if you pass the absolute path to the file to upload.
After submitting the form, we checked that the executed action is create.

Testing the Form
----------------

If the form is valid, the job should have been created and the user redirected to the preview page:

src.Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   public function testJobForm()
   {
       // ...
       $client->followRedirect();
       $this->assertEquals('Ibw\JobeetBundle\Controller\JobController::previewAction', $client->getRequest()->attributes->get('_controller'));
   }

Testing the Database Record
---------------------------

Eventually, we want to test that the job has been created in the database and check that the is_activated column is set to false as the user has not published it yet.

src.Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   public function testJobForm()
   {
       // ...
       $kernel = static::createKernel();
       $kernel->boot();
       $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

       $query = $em->createQuery('SELECT count(j.id) from IbwJobeetBundle:Job j WHERE j.location = :location AND j.is_activated IS NULL AND j.is_public = 0');
       $query->setParameter('location', 'Atlanta, USA');
       $this->assertTrue(0 < $query->getSingleScalarResult());
   }

Testing for Errors
------------------

The job form creation works as expected when we submit valid values. Let’s add a test to check the behavior when we submit non-valid data:

.. code-block:: php

   public function testJobForm()
   {
       // ...
       $crawler = $client->request('GET', '/job/new');
       $form = $crawler->selectButton('Preview your job')->form(array(
           'job[company]'      => 'Sensio Labs',
           'job[position]'     => 'Developer',
           'job[location]'     => 'Atlanta, USA',
           'job[email]'        => 'not.an.email',
       ));
       $crawler = $client->submit($form);

       // check if we have 3 errors
       $this->assertTrue($crawler->filter('.error_list')->count() == 3);

       // check if we have error on job_description field
       $this->assertTrue($crawler->filter('#job_description')->siblings()->first()->filter('.error_list')->count() == 1);

       // check if we have error on job_how_to_apply field
       $this->assertTrue($crawler->filter('#job_how_to_apply')->siblings()->first()->filter('.error_list')->count() == 1);

       // check if we have error on job_email field
       $this->assertTrue($crawler->filter('#job_email')->siblings()->first()->filter('.error_list')->count() == 1);
   }

Now, we need to test the admin bar found on the job preview page. When a job has not been activated yet, you can edit, delete, or publish the job. To test those three actions, we will need to first create a job. But that’s a lot of copy and paste, so let’s add a job creator method in the JobControllerTest class:

src.Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   // ...

   public function createJob($values = array())
   {
       $client = static::createClient();
       $crawler = $client->request('GET', '/job/new');
       $form = $crawler->selectButton('Preview your job')->form(array_merge(array(
           'job[company]'      => 'Sensio Labs',
           'job[url]'          => 'http://www.sensio.com/',
           'job[position]'     => 'Developer',
           'job[location]'     => 'Atlanta, USA',
           'job[description]'  => 'You will work with symfony to develop websites for our customers.',
           'job[how_to_apply]' => 'Send me an email',
           'job[email]'        => 'for.a.job@example.com',
           'job[is_public]'    => false,
     ), $values));

       $client->submit($form);
       $client->followRedirect();

       return $client;
   }

The createJob() method creates a job, follows the redirect and returns the browser. You can also pass an array of values that will be merged with some default values.
Testing the Publish action is now more simple:

src/ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   public function testPublishJob()
   {
       $client = $this->createJob(array('job[position]' => 'FOO1'));
       $crawler = $client->getCrawler();
       $form = $crawler->selectButton('Publish')->form();
       $client->submit($form);

       $kernel = static::createKernel();
       $kernel->boot();
       $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

       $query = $em->createQuery('SELECT count(j.id) from IbwJobeetBundle:Job j WHERE j.position = :position AND j.is_activated = 1');
       $query->setParameter('position', 'FOO1');
       $this->assertTrue(0 < $query->getSingleScalarResult());
   }

Testing the Delete action is quite similar:
src.Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   // ...

   public function testDeleteJob()
   {
       $client = $this->createJob(array('job[position]' => 'FOO2'));
       $crawler = $client->getCrawler();
       $form = $crawler->selectButton('Delete')->form();
       $client->submit($form);

       $kernel = static::createKernel();
       $kernel->boot();
       $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

       $query = $em->createQuery('SELECT count(j.id) from IbwJobeetBundle:Job j WHERE j.position = :position');
       $query->setParameter('position', 'FOO2');
       $this->assertTrue(0 == $query->getSingleScalarResult());
   }

Tests as a SafeGuard
--------------------

When a job is published, you cannot edit it anymore. Even if the “Edit” link is not displayed anymore on the preview page, let’s add some tests for this requirement.
First, add another argument to the createJob() method to allow automatic publication of the job, and create  a getJobByPosition() method that returns a job given its position value:

src.Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   // ...

   public function createJob($values = array(), $publish = false)
   {
       $client = static::createClient();
       $crawler = $client->request('GET', '/job/new');
       $form = $crawler->selectButton('Preview your job')->form(array_merge(array(
           'job[company]'      => 'Sensio Labs',
           'job[url]'          => 'http://www.sensio.com/',
           'job[position]'     => 'Developer',
           'job[location]'     => 'Atlanta, USA',
           'job[description]'  => 'You will work with symfony to develop websites for our customers.',
           'job[how_to_apply]' => 'Send me an email',
           'job[email]'        => 'for.a.job@example.com',
           'job[is_public]'    => false,
     ), $values));

       $client->submit($form);
       $client->followRedirect();

       if($publish) {
         $crawler = $client->getCrawler();
         $form = $crawler->selectButton('Publish')->form();
         $client->submit($form);
         $client->followRedirect();
       }

     return $client;
   }

   public function getJobByPosition($position)
   {
       $kernel = static::createKernel();
       $kernel->boot();
       $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');

       $query = $em->createQuery('SELECT j from IbwJobeetBundle:Job j WHERE j.position = :position');
       $query->setParameter('position', $position);
       $query->setMaxResults(1);
       return $query->getSingleResult();
   }

If a job is published, the edit page must return a 404 status code:

src.Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   // ...

   public function testEditJob()
   {
       $client = $this->createJob(array('job[position]' => 'FOO3'), true);
       $crawler = $client->getCrawler();
       $crawler = $client->request('GET', sprintf('/job/%s/edit', $this->getJobByPosition('FOO3')->getToken()));
       $this->assertTrue(404 === $client->getResponse()->getStatusCode());
   }

But if you run the tests, you won’t have the expected result as we forgot to implement this security measure yesterday. Writing tests is also a great way to discover bugs, as you need to think about all edge cases.
Fixing the bug is quite simple as we just need to forward to a 404 page if the job is activated:

src/Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

   public function editAction($token)
   {
       $em = $this->getDoctrine()->getManager();

       $entity = $em->getRepository('IbwJobeetBundle:Job')->findOneByToken($token);

       if (!$entity) {
           throw $this->createNotFoundException('Unable to find Job entity.');
       }

       if ($entity->getIsActivated()) {
           throw $this->createNotFoundException('Job is activated and cannot be edited.');
       }

     // ...
   }

Back to the Future in a Test
----------------------------

When a job is expiring in less than five days, or if it is already expired, the user can extend the job validation for another 30 days from the current date.
Testing this requirement in a browser is not easy as the expiration date is automatically set when the job is created to 30 days in the future. So, when getting the job page, the link to extend the job is not present. Sure, you can hack the expiration date in the database, or tweak the template to always display the link, but that’s tedious and error prone. As you have already guessed, writing some tests will help us one more time.
As always, we need to add a new route for the extend method first:

src/Ibw/JobeetBundle/Resources/config/routing/job.yml

.. code-block:: yaml

   # ...

   ibw_job_extend:
       pattern:  /{token}/extend
       defaults: { _controller: "IbwJobeetBundle:Job:extend" }
       requirements: { _method: post }

Then, replace the Extend link code in the admin.html.twig partial with the extend form:

src/Ibw/JobeetBundle/Resources/views/Job/admin.html.twig

.. code-block:: html+jinja

   <!-- ... -->

   {% if job.expiresSoon %}
       <form action="{{ path('ibw_job_extend', { 'token': job.token }) }}" method="post">
           {{ form_widget(extend_form) }}
           <button type="submit">Extend</button> for another 30 days
       </form>
   {% endif %}

   <!-- ... -->

Then, create the extend action and the extend form:

src.Ibw/JobeetBundle/Controller/JobController.php

.. code-block:: php

   // ...

   public function extendAction(Request $request, $token)
   {
       $form = $this->createExtendForm($token);
       $request = $this->getRequest();

       $form->bind($request);

       if($form->isValid()) {
           $em=$this->getDoctrine()->getManager();
           $entity = $em->getRepository('IbwJobeetBundle:Job')->findOneByToken($token);

           if(!$entity){
               throw $this->createNotFoundException('Unable to find Job entity.');
           }

           if(!$entity->extend()){
               throw $this->createNodFoundException('Unable to extend the Job');
           }

           $em->persist($entity);
           $em->flush();

           $this->get('session')->getFlashBag()->add('notice', sprintf('Your job validity has been extended until %s', $entity->getExpiresAt()->format('m/d/Y')));
       }

       return $this->redirect($this->generateUrl('ibw_job_preview', array(
           'company' => $entity->getCompanySlug(),
           'location' => $entity->getLocationSlug(),
           'token' => $entity->getToken(),
           'position' => $entity->getPositionSlug()
       )));
   }

   private function createExtendForm($token)
   {
       return $this->createFormBuilder(array('token' => $token))
           ->add('token', 'hidden')
           ->getForm();
   }

Also, add the extend form to the preview action:

src/Ibw/JobeetBundle/Controller/JobController.phpPHP

.. code-block:: php

   // ...

   public function previewAction($token)
   {
       $em = $this->getDoctrine()->getManager();

       $entity = $em->getRepository('IbwJobeetBundle:Job')->findOneByToken($token);

       if (!$entity) {
           throw $this->createNotFoundException('Unable to find Job entity.');
       }

       $deleteForm = $this->createDeleteForm($entity->getId());
       $publishForm = $this->createPublishForm($entity->getToken());
       $extendForm = $this->createExtendForm($entity->getToken());

       return $this->render('IbwJobeetBundle:Job:show.html.twig', array(
           'entity'      => $entity,
           'delete_form' => $deleteForm->createView(),
           'publish_form' => $publishForm->createView(),
           'extend_form' => $extendForm->createView(),
       ));
   }

As expected by the action, the extend() method of Job returns true if the job has been extended or false otherwise:

src/Ibw/JobeetBundle/Entity/Job.php

.. code-block:: php

   // ...

   public function extend()
   {
       if (!$this->expiresSoon())
       {
           return false;
       }

       $this->expires_at = new \DateTime(date('Y-m-d H:i:s', time() + 86400 * 30));

       return true;
   }

Eventually, add a test scenario:

src.Ibw/JobeetBundle/Tests/Controller/JobControllerTest.php

.. code-block:: php

   // ...

   public function testExtendJob()
   {
       // A job validity cannot be extended before the job expires soon
       $client = $this->createJob(array('job[position]' => 'FOO4'), true);
       $crawler = $client->getCrawler();
       $this->assertTrue($crawler->filter('input[type=submit]:contains("Extend")')->count() == 0);

       // A job validity can be extended when the job expires soon

       // Create a new FOO5 job
       $client = $this->createJob(array('job[position]' => 'FOO5'), true);

       // Get the job and change the expire date to today
       $kernel = static::createKernel();
       $kernel->boot();
       $em = $kernel->getContainer()->get('doctrine.orm.entity_manager');
       $job = $em->getRepository('IbwJobeetBundle:Job')->findOneByPosition('FOO5');
       $job->setExpiresAt(new \DateTime());
       $em->flush();

       // Go to the preview page and extend the job
       $crawler = $client->request('GET', sprintf('/job/%s/%s/%s/%s', $job->getCompanySlug(), $job->getLocationSlug(), $job->getToken(), $job->getPositionSlug()));
       $crawler = $client->getCrawler();
       $form = $crawler->selectButton('Extend')->form();
       $client->submit($form);

       // Reload the job from db
       $job = $this->getJobByPosition('FOO5');

       // Check the expiration date
       $this->assertTrue($job->getExpiresAt()->format('y/m/d') == date('y/m/d', time() + 86400 * 30));
   }

Maintenance Tasks
-----------------

Even if symfony is a web framework, it comes with a command line tool. You have already used it to create the default directory structure of the application bundle and to generate various files for the model. Adding a new command is quite easy.
When a user creates a job, he must activate it to put it online. But if not, the database will grow with stale jobs. Let’s create a command that remove stale jobs from the database. This command will have to be run regularly in a cron job.

src/Ibw/JobeetBundle/Command/JobeetCleanupCommand.php

.. code-block:: php

   namespace Ibw\JobeetBundle\Command;

   use Symfony\Bundle\FrameworkBundle\Command\ContainerAwareCommand;
   use Symfony\Component\Console\Input\InputArgument;
   use Symfony\Component\Console\Input\InputInterface;
   use Symfony\Component\Console\Input\InputOption;
   use Symfony\Component\Console\Output\OutputInterface;
   use Ibw\JobeetBundle\Entity\Job;

   class JobeetCleanupCommand extends ContainerAwareCommand {

     protected function configure()
     {
         $this
             ->setName('ibw:jobeet:cleanup')
             ->setDescription('Cleanup Jobeet database')
             ->addArgument('days', InputArgument::OPTIONAL, 'The email', 90)
       ;
     }

     protected function execute(InputInterface $input, OutputInterface $output)
     {
         $days = $input->getArgument('days');

         $em = $this->getContainer()->get('doctrine')->getManager();
         $nb = $em->getRepository('IbwJobeetBundle:Job')->cleanup($days);

         $output->writeln(sprintf('Removed %d stale jobs', $nb));
     }
   }

You will have to add the cleanup method to the JobRepository class:

src/Ibw/JobeetBundle/Repository/JobRepository.php

.. code-block:: php

   // ...

   public function cleanup($days)
   {
       $query = $this->createQueryBuilder('j')
           ->delete()
           ->where('j.is_activated IS NULL')
           ->andWhere('j.created_at < :created_at')
           ->setParameter('created_at',  date('Y-m-d', time() - 86400 * $days))
           ->getQuery();

       return $query->execute();
   }

To run the command execute the following from the project folder:

.. code-block:: bash

   $ php app/console ibw:jobeet:cleanup

or:

.. code-block:: bash

   $ php app/console ibw:jobeet:cleanup 10

to delete stale jobs older than 10 days.


.. include:: common/license.rst.inc
