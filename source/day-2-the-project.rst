2日目: このプロジェクトについて
==============================
.. include:: common/original.rst.inc

私たちは、1日目で、まだコードを一行も書かれていますが、していない、私たちは、セットアップ環境をと、空のsymfonyプロジェクトを作成しました。
この日は、プロジェクトの仕様についてです。コード頭最初に飛び込む前に、もう少しプロジェクトについて説明してみましょう。以下のセクションでは、私たちはいくつかの単純なストーリーでプロジェクトの最初のバージョン/イテレーションで実装する機能について説明します。
We have not written a single line of code yet, but, in Day 1, we setup the environment and created an empty Symfony project.
This day is about the project specifications. Before diving into the code head-first, let’s describe the project a bit more. The following sections describe the features we want to implement in the first version/iteration of the project with some simple stories.

ユーザーストーリー
User Stories
------------

JobeetのWebサイトは、4つのユーザータイプがあります：管理者（ウェブサイトを所有し、管理している）、ユーザー（訪問仕事を探してウェブサイト）、ポスター（訪問ジョブを投稿するウェブサイト）、アフィリエイトを（彼のウェブサイト上のジョブを再発行して）。
元のチュートリアルでは、2つのアプリケーション管理者がウェブサイトを管理するユーザがWebサイトと対話するフロントエンド、バックエンドを作る必要がありました。 symfonyの2.3.2を使用して、私たちはもはやこれをしないだろう。私たちは、その中に、管理者ごとに個別のセキュリティで保護されたセクションを一つだけのアプリケーションを持っているでしょう。
The Jobeet website will have four type of users: admin (owns and manages the website), user (visits the website looking for a job), poster (visits the website to post jobs) and affiliate (re-publishes jobs on his website).
In the original tutorial, we had to make two applications, the frontend, where the users interact with the website, and the backend, where admins manage the website. Using Symfony 2.3.2, we would not do this anymore. We will have only one application and, in it, a separate secured section for admins.

ストーリーF1は：ホームページでは、ユーザーは最新のアクティブなジョブを見ている
Story F1: On the homepage, the user sees the latest active jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザーがJobeetのWebサイトに来るとき、彼はアクティブなジョブのリストを見ている。ジョブは、発行日までにカテゴリ別にしてからソートされている - 新しいジョブを最初に。各ジョブのために、唯一の場所、利用可能な位置および会社が表示されます。
各カテゴリでは、リストは最初の10ジョブと指定されたカテゴリ（ストーリーF2）のためのすべてのジョブを一覧表示することができ、リンクが表示されます。
ホームページでは、ユーザーは、ジョブリスト（ストーリーF3）を絞り込むか、新しいジョブ（ストーリーF5）を投稿することができます。
When a user comes to Jobeet website, he sees a list of active jobs. The jobs are sorted by category and then by publication date – newer jobs first. For each job, only the location, the position available and the company are displayed.
For each category, the list shows the first 10 jobs and a link that allows to list all the jobs for a given category (Story F2).
On the homepage, the user can refine the job list (Story F3) or post a new job (Story F5).

ストーリーF2：ユーザーが特定のカテゴリ内のすべてのジョブを求めることができます
Story F2: A user can ask for all the jobs in a given category
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザーはカテゴリ名やホームページに「雇用」リンクをクリックすると、彼は日付順にソートされ、このカテゴリのすべてのジョブを見ている。
リストはページごとに20件の求人でページ分割されている。
When a user clicks on a category name or on a “more jobs” link on the homepage, he sees all the jobs for this category sorted by date.
The list is paginated with 20 jobs per page.

ストーリーF3：ユーザーは、いくつかのキーワードを含むリストを洗練
Story F3: A user refines the list with some keywords
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ユーザは、自分の検索にいくつかのキーワードを入力することができます。キーワードは、場所、位置、カテゴリまたは会社フィールドにある言葉であることができる。
The user can enter some keywords to refine his search. Keywords can be words found in the location, the position, the category or the company fields.

ストーリーF4：ユーザーは、より詳細な情報を表示するには、ジョブをクリック
Story F4: A user clicks on a job to see more detailed information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザは、より詳細な情報を表示するには、リストからジョブを選択することができます。
The user can select a job from a list to see more detailed information.

ストーリーF5：ユーザーは、ジョブをポスト
Story F5: A user posts a job
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザーは、ジョブを投稿することができます。ジョブは、複数の情報から構成されます。
A user can post a job. A job is made of several pieces of information:

*会社
*タイプ（フルタイム、パートタイムもしくはフリーランス）
*ロゴ（オプション）
* URL（オプション）
*ポジション
*位置
*カテゴリ（ユーザーは、可能なカテゴリのリストで選択されます）
*ジョブの説明（URLとメールが自動的にリンクされます）
*応募方法（URLとメールは自動的にリンクされます）
*パブリック（ジョブもアフィリエイトのウェブサイト上で公開することができ乗り切る）
* Eメール（投稿者の電子メール）

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

プロセスは、2つのステップがあります：まず、ユーザーは、仕事を記述するために、すべての必要な情報をフォームに入力した後、最終的な仕事のページをプレビューすることにより、情報を検証します。
仕事を投稿するACOUNTを作成する必要はありません。ジョブ（ジョブが作成されたときにユーザに与えられたトークンによって保護された）特定のURLへのその後のおかげで変更することができる。
各ジョブポストは30日（これは管理者によって設定される）のためのオンラインです。ユーザーが再度アクティブにするか、余分な30日間の仕事の有効性を拡張するために戻ってくることができますが、場合にのみジョブが5日未満で期限切れとなります。
The process has only two steps: first, the user fills in the form with all the needed information to describe the job, then validates the information by previewing the final job page.
There is no need to create an acount to post a job. A job can be modified afterwards thanks to a specific URL (protected by a token given to the user when the job is created).
Each job post is online for 30 days (this is configurable by admin). A user can come back to re-activate or extend the validity of the job for an extra 30 days, but only when the job expires in less than 5 days.

ストーリーF6：ユーザーがアフィリエイトになるために適用されます
Story F6: A user applies to become an affiliate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ユーザーがアフィリエイトになるためにとJobeetのAPIを使用する権限を適用する必要があります。適用するには、彼は次のような情報を提供する必要があります。
A user needs to apply to become an affiliate and be authorized to use Jobeet API. To apply, he must give the following information:

*名前
* Eメール
*ウェブサイトのURL
*  Name
*  Email
*  Website URL

アフィリエイトのアカウントが管理者（ストーリーB3）によって活性化されなければならない。活性化した後は、関連会社が電子メールを介して、APIで使用するトークンを受け取ります。
The affiliate account must be activated by the admin (Story B3). Once activated, the affiliate receives a token to use with the API via email.

ストーリーF7キー：アフィリエイトは現在アクティブなジョブリストを取得します
Story F7: An affiliate retrieves the current active job list
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

アフィリエイトは、彼のアフィリエイトトークンを使用してAPIを呼び出して、現在のジョブリストを取得できます。リストは、XML、JSONやYAML形式で返すことができます。アフィリエイトは、ジョブの数が返される制限し、また、カテゴリを指定することでクエリを絞り込むことができます。
An affiliate can retrieve the current job list by calling the API with his affiliate token. The list can be returned in the XML, JSON or YAML format. The affiliate can limit the number of jobs to be returned and, also, refine his query by specifying a category.

ストーリーB1：管理者は、Webサイトを構成します
Story B1: An admin configures the website
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

管理者はウェブサイト上でカテゴリが利用可能に編集することができます。
An admin can edit the categories available on the website.

ストーリーB2：管理者は、ジョブを管理
Story B2: An admin manages the jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

管理者は編集し、掲載したジョブを削除することができます。
An admin can edit and remove any posted job.

ストーリーB3：管理者は、関連会社を管理する
Story B3: An admin manages the affiliates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

管理者は、作成または編集の関連会社ができる。彼は、アフィリエイトを活性化させるための責任があり、また、1を無効にすることができます。管理者は新しいアフィリエイトをアクティブにすると、システムは、アフィリエイトが使用する一意のトークンを作成します。
開発者は、初日からコードを書き始めることはありません。まず、あなたは、あなたのプロジェクトの要件を収集する必要が、あなたのプロジェクトが動作するようになっているかを理解しています。それはあなたが今日やっていることだ。また明日！
The admin can create or edit affiliates. He is responsible for activating an affiliate and can also disable one. When the admin activates a new affiliate, the system creates a unique token to be used by the affiliate.
As a developer, you never start coding from the first day. Firstly, you need to gather the requirements of your project and understand how your project is supposed to work. That’s what you have done today. See you tomorrow!

.. include:: common/license.rst.inc
