Day 5: The Routing
==================
* This article is part of the original Jobeet Tutorial, created by Fabien Potencier, for Symfony 1.4.
URLs

If you click on a job on the Jobeet homepage, the URL looks like this: /job/1/show. If you have already developed PHP websites, you are probably more accustomed to URLs like /job.php?id=1. How does Symfony make it work? How does Symfony determine the action to call based on this URL? Why is the id of the job retrieved with the $id parameter in the action? Here, we will answer all these questions.
You have already seen the following code in the src/Ibw/JobeetBundle/Resources/views/Job/index.html.twig template:

1
{{ path('ibw_job_show', { 'id': entity.id }) }}
This uses the path template helper function to generate the url for the job which has the id 1. The ibw_job_show is the name of the route used, defined in the configuration as you will see below.
Routing Configuration

In Symfony2, routing configuration is usually done in the app/config/routing.yml. This imports specific bundle routing configuration. In our case, the src/Ibw/JobeetBundle/Resources/config/routing.yml file is imported:
app/config/routing.ymlYAML

ibw_jobeet:
    resource: "@IbwJobeetBundle/Resources/config/routing.yml"
    prefix:   /
Now, if you look in the JobeetBundle routing.yml you will see that it imports another routing file, the one for the Job controller and defines a route called ibw_jobeet_homepage for the /hello/{name} URL pattern:
src/Ibw/JobeetBundle/Resources/config/routing.ymlYAML

IbwJobeetBundle_job:
    resource: "@IbwJobeetBundle/Resources/config/routing/job.yml"
    prefix: /job

ibw_jobeet_homepage:
    pattern:  /hello/{name}
    defaults: { _controller: IbwJobeetBundle:Default:index }
src/Ibw/JobeetBundle/Resources/config/routing/job.ymlYAML

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
Let’s have a closer look to the ibw_job_show route. The pattern defined by the ibw_job_show route acts like /*/show where the wildcard is given the name id. For the URL /1/show, the id variable gets a value of 1, which is available for you to use in your controller. The _controller parameter is a special key that tells Symfony which controller/action should be executed when a URL matches this route, in our case it should execute the showAction from the JobController in the IbwJobeetBundle.
The route parameters (e.g. {id}) are especially important because each is made available as an argument to the controller method.
Routing Configuration in Dev Environment

The dev environment loads the app/config/routing_dev.yml file that contains the routes used by the Web Debug Toolbar (you already deleted the routes for the AcmeDemoBundle from /app/config/routing_dev.php – see Day 1, How to remove the AcmeDemoBundle). This file loads, at the end, the main routing.yml configuration file.
Route Customizations

For now, when you request the / URL in a browser, you will get a 404 Not Found error. That’s because this URL does not match any routes defined. We have a  ibw_jobeet_homepage route that matches the /hello/jobeet URL and sends us to the DefaultController, index action. Let’s change it to match the / URL and to call the index action from the JobController. To make the change, modify it to the following:
src/Ibw/JobeetBundle/Resources/config/routing.ymlYAML

# ...
ibw_jobeet_homepage:
    pattern:  /
    defaults: { _controller: IbwJobeetBundle:Job:index }
Now, if you clear the cache and go to http://jobeet.local from your browser, you will see the Job homepage. We can now change the link of the Jobeet logo in the layout to use the ibw_jobeet_homepage route:
src/Ibw/JobeetBundle/Resources/views/layout.html.twigXHTML

<!-- ... -->
    <h1><a href="{{ path('ibw_jobeet_homepage') }}">
        <img alt="Jobeet Job Board" src="{{ asset('bundles/ibwjobeet/images/logo.jpg') }}" />
    </a></h1>
<!-- ... -->
For something a bit more involved, let’s change the job page URL to something more meaningful:

1
/job/sensio-labs/paris-france/1/web-developer
Without knowing anything about Jobeet, and without looking at the page, you can understand from the URL that Sensio Labs is looking for a Web developer to work in Paris, France.
The following pattern matches such a URL:

1
/job/{company}/{location}/{id}/{position}
Edit the ibw_job_show route from the job.yml file:
src/Ibw/JobeetBundle/Resources/config/routing/job.ymlYAML

# ...

ibw_job_show:
    pattern:  /{company}/{location}/{id}/{position}
    defaults: { _controller: "IbwJobeetBundle:Job:show" }
Now, we need to pass all the parameters for the changed route for it to work:
src/Ibw/JobeetBundle/Resources/views/Job/index.html.twigXHTML

<!-- ... -->
<a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.company, 'location': entity.location, 'position': entity.position }) }}">
    {{ entity.position }}
</a>
<!-- ... -->
If you have a look at generated URLs, they are not quite yet as we want them to be:
http://jobeet.local/app_dev.php/job/Sensio Labs/Paris,France/1/Web Developer
We need to “slugify” the column values by replacing all non ASCII characters by a -. Open the Job.php file and add the following methods to the class:
src/Ibw/JobeetBundle/Entity/Job.phpPHP

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

You must also add the use statement before the Job class definition.
After that, create the src/Ibw/JobeetBundle/Utils/Jobeet.php file and add the slugify method in it:
src/Ibw/JobeetBundle/Utils/Jobeet.phpPHP

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

We have defined three new “virtual” accessors: getCompanySlug(), getPositionSlug(), and getLocationSlug(). They return their corresponding column value after applying it the slugify() method. Now, you can replace the real column names by these virtual ones in the template:
src/Ibw/JobeetBundle/views/Job/index.html.twigXHTML

<!-- ... -->
<a href="{{ path('ibw_job_show', { 'id': entity.id, 'company': entity.companyslug, 'location': entity.locationslug, 'position': entity.positionslug}) }}">
   {{ entity.position }}
</a>
<!-- ... -->
Route Requirements

The routing system has a built-in validation feature. Each pattern variable can be validated by a regular expression defined using the requirements entry of a route definition:
src/Ibw/JobeetBundle/Resources/config/routing/job.ymlYAML

# ...
ibw_job_show:
    pattern:  /{company}/{location}/{id}/{position}
    defaults: { _controller: "IbwJobeetBundle:Job:show" }
    requirements:
        id:  \d+

# ...
The above requirements entry forces the id to be a numeric value. If not, the route won’t match.
Route Debugging

While adding and customizing routes, it’s helpful to be able to visualize and get detailed information about your routes. A great way to see every route in your application is via the router:debug console command. Execute the command by running the following from the root of your project:

1
php app/console router:debug
The command will print a helpful list of all the configured routes in your application. You can also get very specific information on a single route by including the route name after the command:

1
php app/console router:debug ibw_job_show
Final Thoughts

That’s all for today! To learn more about the Symfony2 routing system read the Routing chapter form the book.
Creative Commons License
This work is licensed under a Creative Commons Attribution-ShareAlike 3.0 Unported License.
