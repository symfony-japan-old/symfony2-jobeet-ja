Day 15: Web Services
====================

* This article is part of the original Jobeet Tutorial, created by Fabien Potencier, for Symfony 1.4.
With the addition of feeds on Jobeet, job seekers can now be informed of new jobs in real-time.
On the other side of the fence, when you post a job, you will want to have the greatest exposure possible. If your job is syndicated on a lot of small websites, you will have a better chance to find the right person. That’s the power of the long tail. Affiliates will be able to publish the latest posted jobs on their websites thanks to the web services we will develop today.
Affiliates

As we already said in day 2 of this tutorial, an affiliate retrieves the current active job list.
The fixtures

Let’s create a new fixture file for the affiliates:
src/Ibw/JobeetBundle/DataFixtures/ORM/LoadAffiliateData.phpPHP

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
Now, to persist the data defined in your fixture file, just run the following command:

1
php app/console doctrine:fixtures:load
In the fixture file, the tokens are hardcoded to simplify the testing, but when an actual user applies for an account, the token will need to be generated Let’s create a function to do that in our Affiliate class. Start by adding the setTokenValue method to lifecycleCallbacks section, inside your ORM file:
src//Ibw/JobeetBundle/Resources/config/doctrine/Affiliate.orm.ymlYAML

# ...
    lifecycleCallbacks:
        prePersist: [ setCreatedAtValue, setTokenValue ]
Now, the setTokenValue method will be generated inside the entity file when you will run the following command:

1
php app/console doctrine:generate:entities IbwJobeetBundle
Let’s modify the method now:
src/Ibw/JobeetBundle/Entity/Affiliate.phpPHP

    public function setTokenValue()
    {
        if(!$this->getToken()) {
            $token = sha1($this->getEmail().rand(11111, 99999));
            $this->token = $token;
        }

        return $this;
    }
Reload the data:

1
php app/console doctrine:fixtures:load
The Job Web Service

As always, when you create a new resource, it’s a good habbit to define the route first:
src/Ibw/JobeetBundle/Resources/config/routing.ymlYAML

IbwJobeetBundle_api:
    pattern: /api/{token}/jobs.{_format}
    defaults: {_controller: "IbwJobeetBundle:Api:list"}
    requirements:
        _format: xml|json|yaml
As usually, after you modify a routing file, you need to clear the cache:

php app/console cache:clear --env=dev
php app/console cache:clear --env=prod

The next step is to create the api action and the templates, that will share the same action. Let us now create a new controller file, called ApiController:
src/Ibw/JobeetBundle/Controller/ApiController.php

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
To retrieve the affiliate using his token, we will create the getForToken() method. This method also verifies if the affiliate account is activated, so there is no need for us to check this one more time. Until now, we haven’t used the AffiliateRepository yet, so it doesn’t exist. To create it, modify the ORM file as following, then run the command you used before to generate the entities.
src/Ibw/JobeetBundle/Resources/config/doctrine/Affiliate.orm.ymlYAML

Ibw\JobeetBundle\Entity\Affiliate:
    type: entity
    repositoryClass: Ibw\JobeetBundle\Repository\AffiliateRepository
    # ...
Once created, it is ready to be used:
src/Ibw/JobeetBundle/Repository/AffiliateRepository.phpPHP

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
After identifying the affiliate by his token, we will use the getActiveJobs() method to give the affiliate the jobs he required, belonging to the selected categories. If you open your JobRepository file now, you will see that the getActiveJobs() method doesn’t share any connection with the affiliates. Because we want to reuse that method, we need to make some modifications inside of it:
src/Ibw/JobeetBundle/Repository/JobRepository.phpPHP

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
As you can see, we populate the jobs array using a function called asArray(). Let’s define it:
src/Ibw/JobeetBundle/Entity/Job.phpPHP

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

Supporting the xml format is as simple as creating a template:
src/Ibw/JobeetBundle/Resources/views/Api/jobs.xml.twigXHTML

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

Support the JSON format is similar:
src/Ibw/JobeetBundle/Resources/views/Api/jobs.json.twigXHTML

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

src/Ibw/JobeetBundle/Resources/views/Api/jobs.yaml.twigYAML

{% for url,job in jobs %}
    Url: {{ url }}
{% for key, value in job %}
        {{ key }}: {{ value }}
{% endfor %}
{% endfor %}
If you try to call the web service with a non-valid token, you will receive a 404 page as a response, for all the formats. To see what you accomplished until now, access the following links: http://jobeet.local/app_dev.php/api/sensio-labs/jobs.xml or http://jobeet.local/app_dev.php/api/symfony/jobs.xml. Change the extension in the URL, depending on which format you prefer.
Web Service Tests

src/Ibw/JobeetBundle/Tests/Controller/ApiControllerTest.phpPHP

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
Inside the ApiControllerTest file, we test that the request formats are correctly received and the pages requested are correctly returned.
The Affiliate Application Form

Now that the web service is ready to be used, let’s create the account creation form for affiliates. For that, you need to write the HTML form, implement validation rules for each field, process the values to store them in a database, display error messages and repopulate fields in case of errors.
First, create a new controller file, named AffiliateController:
src/Ibw/JobeetBundle/Controller/AffiliateController.phpPHP

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
Then, change the Affiliates link in the layout:
src/Ibw/JobeetBundle/Resources/views/layout.html.twigXHTML

<!-- ... -->
    <li class="last"><a href="{{ path('ibw_affiliate_new') }}">Become an affiliate</a></li>
<!-- ... -->
Now, we need to create an action to match the route from the link you just modified it earlier:
src/Ibw/JobeetBundle/Controller/AffiliateController.phpPHP

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
We have the name of the route, we have the action, but we do not have the route. so let’s create it:
src/Ibw/JobeetBundle/Resources/config/routing/affiliate.ymlYAML

ibw_affiliate_new:
    pattern:  /new
    defaults: { _controller: "IbwJobeetBundle:Affiliate:new" }
Also, add this to your routing file:
src/Ibw/JobeetBundle/Resources/config/routing.yml

# ...

IbwJobeetBundle_ibw_affiliate:
    resource: "@IbwJobeetBundle/Resources/config/routing/affiliate.yml"
    prefix:   /affiliate
The form file also needs to be created. But, even if the Affiliate has more fields, we won’t display them all, because some of them must not be editable by the end user. Create your Affiliate form:
src/Ibw/JobeetBundle/Form/AffiliateType.phpPHP

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
Now, we need to decide whether or not the Affiliate object is valid after the form has applied the submitted data to it. To do this, add the following code to your validation file:
src/Ibw/JobeetBundle/Resources/config/validation.ymlYAML

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
In the validation schema, we used a new validator, called UniqueEntity.  It validates that a particular field (or fields) in a Doctrine entity is (are) unique. This is commonly used, for example, to prevent a new user to register using an email address that already exists in the system.
Don’t forget to clear your cache after applying the validation constraints!
Finally, let’s create the view for the form too:
src/Ibw/JobeetBundle/Resources/views/Affiliate/affiliate_new.html.twigXHTML

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
When the user submits a form, the form data must be persisted into database, if valid. Add the new create action to your Affiliate controller:
src/Ibw/JobeetBundle/Controller/AffiliateController.phpPHP

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
When submitting, the create action is performed, so we need to define the route:
src/Ibw/JobeetBundle/Resources/config/routing/affiliate.ymlYAML

# ...

ibw_affiliate_create:
    pattern: /create
    defaults: { _controller: "IbwJobeetBundle:Affiliate:create" }
    requirements: { _method: post }
After the affiliate registers, he is redirected to a waiting page. Let’s define that action and create the view too:
src/Ibw/JobeetBundle/Controller/AffiliateController.phpPHP

class AffiliateController extends Controller
{
    // ...

    public function waitAction()
    {
        return $this->render('IbwJobeetBundle:Affiliate:wait.html.twig');
    }
}
src/Ibw/JobeetBundle/Resources/views/Affiliate/wait.html.twigXHTML

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
Now, the route:
src/Ibw/JobeetBundle/Resources/config/routing/affiliate.ymlYAML

# ...

ibw_affiliate_wait:
    pattern: /wait
    defaults: { _controller: "IbwJobeetBundle:Affiliate:wait" }
After defining to routes, in order to work, you need to clear the cache.
Now, if you click on the Affiliates link on the homepage, you will be directed to the affiliate form page.
Tests

The last step is to write some functional tests for the new feature.
src/Ibw/JobeetBundle/Tests/Controller/AffiliateControllerTest.phpPHP

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

For the backend, we will work with SonataAdminBundle. As we said before, after an affiliate registers, he needs to wait for the admin to activate his account. So, when the admin will access the affiliates page, he will see only the inactivated accounts, to help him be more productive.
First of all, you need to declare the new affiliate service inside your services.yml file:
src/Ibw/JobeetBundle/Resources/config/services.ymlYAML

# ...
    ibw.jobeet.admin.affiliate:
        class: Ibw\JobeetBundle\Admin\AffiliateAdmin
        tags:
            - { name: sonata.admin, manager_type: orm, group: jobeet, label: Affiliates }
        arguments:
            - ~
            - Ibw\JobeetBundle\Entity\Affiliate
            - 'IbwJobeetBundle:AffiliateAdmin'
After that, create the Admin file:
src/Ibw/JobeetBundle/Admin/AffiliateAdmin.phpPHP

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
To help the administrator, we want to display only the inactivated accounts. This can be made by setting the ‘is_active’ filter to false:
src/Ibw/JobeetBundle/Admin/AffiliateAdmin.phpPHP

// ...
    protected $datagridValues = array(
        '_sort_order' => 'ASC',
        '_sort_by' => 'is_active',
        'is_active' => array('value' => 2) // The value 2 represents that the displayed affiliate accounts are not activated yet
    );

// ...
Now, create the AffiliateAdmin controller file:
src/Ibw/JobeetBundle/Controller/AffiliateAdminController.phpPHP

namespace Ibw\JobeetBundle\Controller;

use Sonata\AdminBundle\Controller\CRUDController as Controller;
use Sonata\DoctrineORMAdminBundle\Datagrid\ProxyQuery as ProxyQueryInterface;
use Symfony\Component\HttpFoundation\RedirectResponse;

class AffiliateAdminController extends Controller
{
    // Your code goes here
}
Let’s create the activate and deactivate batch actions:
src/Ibw/JobeetBundle/Controller/AffiliateAdminController.phpPHP

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
For the new batch actions to be functional, we have to add them in the getBatchActions from the Admin class:
src/Ibw/JobeetBundle/Admin/AffiliateAdmin.phpPHP

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
For this to work, you need to add the two methods, activate and deactivate, in the entity file:
src/Ibw/JobeetBundle/Entity/Affiliate.phpPHP

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
Let’s now create two individual actions, activate and deactivate, for each item. Firstly, we will create routes for them. That’s why, in your Admin class, you will extend the configureRoutes function:
src/Ibw/JobeetBundle/Admin/AffiliateAdmin.phpPHP

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
It’s time to implement the actions in the AdminController:
src/Ibbw/JobeetBundle/Controller/AffiliateAdminController.phpPHP

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
Now, create the templates for the new added action buttons:
src/Ibw/JobeetBundle/Resources/views/AffiliateAdmin/list__action_activate.html.twigXHTML

{% if admin.isGranted('EDIT', object) and admin.hasRoute('activate') %}
    <a href="{{ admin.generateObjectUrl('activate', object) }}" class="btn edit_link" title="{{ 'action_activate'|trans({}, 'SonataAdminBundle') }}">
        <i class="icon-edit"></i>
        {{ 'activate'|trans({}, 'SonataAdminBundle') }}
    </a>
{% endif %}
src/Ibw/JobeetBundle/Resources/views/AffiliateAdmin/list__action_deactivate.html.twigXHTML

{% if admin.isGranted('EDIT', object) and admin.hasRoute('deactivate') %}
    <a href="{{ admin.generateObjectUrl('deactivate', object) }}" class="btn edit_link" title="{{ 'action_deactivate'|trans({}, 'SonataAdminBundle') }}">
        <i class="icon-edit"></i>
        {{ 'deactivate'|trans({}, 'SonataAdminBundle') }}
    </a>
{% endif %}
Inside your Admin file, add the new actions and buttons to the configureListFields function, so that they would appear on the page, to each account individually:
src/Ibw/JobeetBundle/Admin/AffiliateAdmin.phpPHP

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
Now, clear your cache and try it on!
That’s all for today! Tomorrow, we will take care of the emails the affiliates will receive when their accounts have been activated.
Creative Commons License
This work is licensed under a Creative Commons Attribution-ShareAlike 3.0 Unported License.
