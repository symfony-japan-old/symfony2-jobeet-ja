2日目: このプロジェクトについて
==============================
.. include:: common/original.rst.inc

We have not written a single line of code yet, but, in Day 1, we setup the environment and created an empty Symfony project.
This day is about the project specifications. Before diving into the code head-first, let’s describe the project a bit more. The following sections describe the features we want to implement in the first version/iteration of the project with some simple stories.

User Stories
------------

The Jobeet website will have four type of users: admin (owns and manages the website), user (visits the website looking for a job), poster (visits the website to post jobs) and affiliate (re-publishes jobs on his website).
In the original tutorial, we had to make two applications, the frontend, where the users interact with the website, and the backend, where admins manage the website. Using Symfony 2.3.2, we would not do this anymore. We will have only one application and, in it, a separate secured section for admins.

Story F1: On the homepage, the user sees the latest active jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a user comes to Jobeet website, he sees a list of active jobs. The jobs are sorted by category and then by publication date – newer jobs first. For each job, only the location, the position available and the company are displayed.
For each category, the list shows the first 10 jobs and a link that allows to list all the jobs for a given category (Story F2).
On the homepage, the user can refine the job list (Story F3) or post a new job (Story F5).

Story F2: A user can ask for all the jobs in a given category
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a user clicks on a category name or on a “more jobs” link on the homepage, he sees all the jobs for this category sorted by date.
The list is paginated with 20 jobs per page.

Story F3: A user refines the list with some keywords
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The user can enter some keywords to refine his search. Keywords can be words found in the location, the position, the category or the company fields.

Story F4: A user clicks on a job to see more detailed information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The user can select a job from a list to see more detailed information.

Story F5: A user posts a job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A user can post a job. A job is made of several pieces of information:
*  Company
*  Type (full-time, part-time or freelance)
*  Logo (optional)
*  URL (optional)
*  Position
*  Location
*  Category (the user chooses in a list of possible categories)
*  Job description (URLs and emails are automatically linked)
*  How to apply (URLs and emails are automatically linked)
*  Public (wether the job can also be published on affiliate websites)
*  Email (email of poster)

The process has only two steps: first, the user fills in the form with all the needed information to describe the job, then validates the information by previewing the final job page.
There is no need to create an acount to post a job. A job can be modified afterwards thanks to a specific URL (protected by a token given to the user when the job is created).
Each job post is online for 30 days (this is configurable by admin). A user can come back to re-activate or extend the validity of the job for an extra 30 days, but only when the job expires in less than 5 days.

Story F6: A user applies to become an affiliate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A user needs to apply to become an affiliate and be authorized to use Jobeet API. To apply, he must give the following information:
*  Name
*  Email
*  Website URL

The affiliate account must be activated by the admin (Story B3). Once activated, the affiliate receives a token to use with the API via email.

Story F7: An affiliate retrieves the current active job list
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An affiliate can retrieve the current job list by calling the API with his affiliate token. The list can be returned in the XML, JSON or YAML format. The affiliate can limit the number of jobs to be returned and, also, refine his query by specifying a category.

Story B1: An admin configures the website
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An admin can edit the categories available on the website.

Story B2: An admin manages the jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An admin can edit and remove any posted job.

Story B3: An admin manages the affiliates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The admin can create or edit affiliates. He is responsible for activating an affiliate and can also disable one. When the admin activates a new affiliate, the system creates a unique token to be used by the affiliate.
As a developer, you never start coding from the first day. Firstly, you need to gather the requirements of your project and understand how your project is supposed to work. That’s what you have done today. See you tomorrow!

.. include:: common/license.rst.inc
