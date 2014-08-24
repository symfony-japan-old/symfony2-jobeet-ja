2日目: このプロジェクトについて
==============================
.. include:: common/original.rst.inc

1日目は、まだコードを一行も書いていませんが、環境セットアップと、空の Symfony プロジェクトを作成しました。
今日は、プロジェクトの仕様についてです。最初にコードに飛び込む前に、もう少しプロジェクトについて説明してみましょう。
以下のセクションでは、いくつかの単純なストーリーでプロジェクトの最初のバージョン/イテレーションで実装する機能について説明します。
We have not written a single line of code yet, but, in Day 1, we setup the environment and created an empty Symfony project.
This day is about the project specifications. Before diving into the code head-first, let’s describe the project a bit more.
The following sections describe the features we want to implement in the first version/iteration of the project with some simple stories.

ユーザーストーリー
------------------

Jobeet の Web サイトは、4つのユーザータイプがあります：管理者（ウェブサイトを所有し、管理している）、ユーザー（仕事を探してウェブサイトを訪問）、投稿者（求人を投稿する為にウェブサイトを訪問）、アフィリエイト（彼のウェブサイト上で求人を再発行）。
元のチュートリアルでは、2つのアプリケーションを作る必要がありました。ユーザが Web サイトと対話するフロントエンドと管理者がウェブサイトを管理するバックエンドです。私たちはもはや Symfony 2.3.2 を使用してこれをしないでしょう。
一つだけのアプリケーションを持ち、その中に管理者のために隔離されセキュリティで保護されたセクションを持つでしょう。
The Jobeet website will have four type of users: admin (owns and manages the website), user (visits the website looking for a job), poster (visits the website to post jobs) and affiliate (re-publishes jobs on his website).
In the original tutorial, we had to make two applications, the frontend, where the users interact with the website, and the backend, where admins manage the website.
 Using Symfony 2.3.2, we would not do this anymore. We will have only one application and, in it, a separate secured section for admins.

ストーリー F1： ホームページでは、ユーザーは最新の有効な求人を見ている
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザーが Jobeet の Web サイトに来るとき、彼はアクティブな求人のリストを見ています。求人は、カテゴリ別に並べられ、且つ、新しい求人を最初にして公開日順に並べられています。
それぞれの求人は場所、可能な役職、および、会社名のみが表示されます。
各カテゴリで、リストには最初の10個の求人が表示されます。そして、指定されたカテゴリ（ストーリー F2 ）のすべての求人を表示する一つのリンクを表示します。
ホームページでは、ユーザーは、求人リスト（ストーリーF3）を絞り込むか、新しい求人（ストーリー F5 ）を投稿することができます。
When a user comes to Jobeet website, he sees a list of active jobs. The jobs are sorted by category and then by publication date – newer jobs first.
For each job, only the location, the position available and the company are displayed.
For each category, the list shows the first 10 jobs and a link that allows to list all the jobs for a given category (Story F2).
On the homepage, the user can refine the job list (Story F3) or post a new job (Story F5).

ストーリー F2：ユーザーが特定のカテゴリ内のすべてのジョブを問い合わせることができます
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザーはカテゴリ名やホームページの「more jobs」リンクをクリックすると、日付順に並べられたこのカテゴリのすべての求人を見ることが出来ます。
リストはページごとに20件の求人でページ分割されています。
When a user clicks on a category name or on a “more jobs” link on the homepage, he sees all the jobs for this category sorted by date.
The list is paginated with 20 jobs per page.

ストーリー F3：ユーザーは複数のキーワードでリストを絞込み
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザはいくつかのキーワードで絞込み検索できます。場所、役職、カテゴリ、会社名欄にある単語をキーワードとすることが出来ます。
The user can enter some keywords to refine his search. Keywords can be words found in the location, the position, the category or the company fields.

ストーリー F4：ユーザーはより詳細な情報を表示する為、求人をクリックします
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザは、より詳細な情報を表示する為に、リストから求人を選択することができます。
The user can select a job from a list to see more detailed information.

ストーリー F5：ユーザーは、求人を投稿します
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザーは、求人を投稿することができます。求人は、複数の情報から構成されます。
A user can post a job. A job is made of several pieces of information:

* 会社
* タイプ（フルタイム、パートタイムもしくはフリーランス）
* ロゴ（オプション）
* URL（オプション）
* 役職
* 場所
* カテゴリ（ユーザーは、選択可能なカテゴリのリストから選びます）
* 仕事の説明（URLとメールが自動的にリンクされます）
* 応募方法（URLとメールは自動的にリンクされます）
* 公開（アフィリエイトのウェブサイト上でも求人を公開することができます）
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

手続きは2つのプロセスのみです。まず、ユーザーは仕事を説明するために、必要なすべての情報をフォームに入力した後、最終的な求人ページをプレビューすることにより、情報を検証します。
仕事を投稿するアカウントを作成する必要はありません。
求人は（作成されたときにユーザに与えられたトークンによって保護された）特定のURLのみで変更することが出来ます。
各求人は30日間有効です（これは管理者によって設定されます）。
ユーザーは戻ってきて再度アクティブにするか、5日未満で期限切れとなる場合のみ、求人の期間の検証を30日以上に広げることが出来ます。

The process has only two steps: first, the user fills in the form with all the needed information to describe the job, then validates the information by previewing the final job page.
There is no need to create an acount to post a job.
A job can be modified afterwards thanks to a specific URL (protected by a token given to the user when the job is created).
Each job post is online for 30 days (this is configurable by admin).
A user can come back to re-activate or extend the validity of the job for an extra 30 days, but only when the job expires in less than 5 days.

ストーリーF6：ユーザーがアフィリエイトになるために適用すること
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ユーザーがアフィリエイトになり、且つ、 Jobeet API を使用することを承認されるために、次のような情報を提供する必要があります。
A user needs to apply to become an affiliate and be authorized to use Jobeet API. To apply, he must give the following information:

*  名前
*  Eメール
*  ウェブサイトのURL
*  Name
*  Email
*  Website URL

アフィリエイトのアカウントは管理者（ストーリー B3 ）によって有効化されなければなりません。有効化した後は、アフィリエイトは API で使用するトークンを電子メールで受け取ります。
The affiliate account must be activated by the admin (Story B3). Once activated, the affiliate receives a token to use with the API via email.

ストーリー F7：アフィリエイトは現在有効な求人のリストを取得します
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

アフィリエイトは、彼のアフィリエイトトークンを使用してAPIを呼び出して、現在の求人リストを取得できます。リストは、 XML 、 JSON や YAML 形式で返すことができます。
アフィリエイトは、返される求人数を制限することが出来、また、カテゴリを指定することでクエリを絞り込むことができます。
An affiliate can retrieve the current job list by calling the API with his affiliate token. The list can be returned in the XML, JSON or YAML format.
The affiliate can limit the number of jobs to be returned and, also, refine his query by specifying a category.

ストーリー B1：管理者はウェブサイトを設定します
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

管理者はカテゴリを編集し、ウェブサイト上で有効にできます。
An admin can edit the categories available on the website.

ストーリー B2：管理者はジョブを管理
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

管理者は投稿された求人を編集、削除することができます。
An admin can edit and remove any posted job.

ストーリー B3：管理者は、関連会社を管理する
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

管理者は、アフィリエイトの作成または編集ができます。彼は、アフィリエイトを有効化する責任があり、また、それを無効にすることができます。
管理者は新しいアフィリエイトを有効にすると、システムは、アフィリエイトが使用する一意のトークンを作成します。
開発者は、初日からコードを書き始めることはありません。まず、あなたはプロジェクトの要件を収集する必要があり、プロジェクトがどのように動作するようになっているかを理解する必要があります。
今日はここまでです。では、また明日！
The admin can create or edit affiliates. He is responsible for activating an affiliate and can also disable one.
When the admin activates a new affiliate, the system creates a unique token to be used by the affiliate.
As a developer, you never start coding from the first day. Firstly, you need to gather the requirements of your project and understand how your project is supposed to work.
That’s what you have done today. See you tomorrow!

.. include:: common/license.rst.inc
