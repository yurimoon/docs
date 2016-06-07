==========
 フォーム
==========

.. php:namespace:: Cake\View\Helper

.. php:class:: FormHelper(View $view, array $config = [])

フォームを作成することにおいて、FormHelperは最も困難な仕事をします。
FormHelperは、バリデーション、再増殖、そしてレイアウトを合理化するやりかたで、素早くフォームを作成することにフォーカスしています。
FormHelperは柔軟でもあります。従来のやり方を使ってもほぼすべてのことができますし、必要な部分だけを取得するための特定のメソッドを使うことができます。

フォームを開始する
==================

.. php:method:: create(mixed $model = null, array $options = [])

FormHelperを活用するために使う必要がある最初のメソッドは、 ``create()`` です。このメソッドはformの開始タグを出力します。

すべてのパラメータは任意です。 ``create()`` をパラメータなしで呼び出したら、現在のURLを通して、現在のコントローラに送信するフォームを作成します。フォーム送信のデフォルトのメソッドはPOSTです。
UsersController::add() のviewの中で ``create()`` を呼び出した場合、以下のような出力をレンダリングされたViewの中で確認できます。::

.. code-block:: html

    <form method="post" action="/users/add">

``$model`` 引数はフォームの 'コンテキスト' として使われます。ビルトインのフォームコンテキストはいくつかあり、自分で足すことができます。そしてそれは次のセクションをカバーします。ビルトインは ``$model`` を以下のようにマッピングします。

* ``Entity`` インスタンス、または ``EntityContext`` のイテレーターマップによって、FormHelperがビルトインORMからの結果を出力できるようになります。
* ``schema`` キーを含む配列は、フォームを構築するシンプルなデータ構造を作成できるようにする ``ArrayContext`` にマッピングします。
* ``null`` および ``false`` は ``NullContext`` にマッピングします。このコンテキストクラスは FormHelper が必要とするインタフェースを満たします。
  このコンテキストは、ORMの存続を必要としない短いフォームを作成したいときに有用です。
すべてのコンテキストクラスは、リクエストデータへのアクセスもできます。このことによって、フォームを作成するのがより簡単になります。

いったんフォームがコンテキストを含んで作成されると、作成したすべての入力はアクティブなコンテキストとして利用されます。
ORMがフォームを返した場合、 FormHelper は関連するデータ、バリデーションエラー、そしてスキーマのメタデータにアクセスできます。
``end()`` メソッドを利用するか、 ``create()`` を再び呼び出すかで、アクティブなコンテキストを閉じることができます。
あるエンティティのフォームを作成したい場合、以下のようにします。::

    // If you are on /articles/add
    // $article should be an empty Article entity.
    echo $this->Form->create($article);

出力:

.. code-block:: html

    <form method="post" action="/articles/add">

これは ArticlesController の ``add()`` アクションにフォームデータを POST します。
しかしながら、 edit のフォームを作るときにも同じロジックを使用することもできます。
FormHelperは add または edit のフォームのどちらを作成するか、を自動的に決定するために、 ``Entity`` オブジェクトを使用します。
提供されたエンティティが 'new' ではなかった場合、フォームは edit のフォームとして作成されます。
例えば、 **http://example.org/articles/edit/5** を閲覧するとき、以下のようにします。::

    // src/Controller/ArticlesController.php:
    public function edit($id = null)
    {
        if (empty($id)) {
            throw new NotFoundException;
        }
        $article = $this->Articles->get($id);
        // Save logic goes here
        $this->set('article', $article);
    }

    // View/Articles/edit.ctp:
    // Since $article->isNew() is false, we will get an edit form
    <?= $this->Form->create($article) ?>

出力:

.. code-block:: html

    <form method="post" action="/articles/edit/5">
    <input type="hidden" name="_method" value="PUT" />

.. note::

    これが edit のフォームである場合、デフォルトの HTTP メソッドを上書きしてhidden の入力フィールドが作成されます。

``$options`` 配列は、フォームの定義が最もなされる場所です。
この特別な配列は、フォームタグを生成するやり方に影響する、異なる key-value ペアがいくつか含まれます。


フォームの HTTP メソッドを変更する
----------------------------------

``type`` オプションを用いることで、フォームで使われるHTTPメソッドを変更することができます。

    echo $this->Form->create($article, ['type' => 'get']);

出力:

.. code-block:: html

    <form method="get" action="/articles/edit/5">

'file' を指定すると、フォームの送信方式は 'post' に変わり、フォームタグには "multipart/form-data" の enctype が含まれます。
これはフォームの中にファイルのエレメントが存在するときに使用されます。
適切な enctype 属性がないときは、ファイルアップロードの機能がなくなります。::

    echo $this->Form->create($article, ['type' => 'file']);

出力:

.. code-block:: html

   <form enctype="multipart/form-data" method="post" action="/articles/add">

'put', 'patch' あるいは 'delete' を用いた場合、フォームは機能的に 'post' のフォームと同等ですが、送信されるときにHTTPリクエストメソッドは 'PUT', 'PATCH' あるいは 'DELETE' にそれぞれ上書きされます。
このことにより CakePHP がウェブブラウザでサポートされている適切な REST をエミュレートすることができます。

フォームにURLを設定する
-----------------------

``url`` オプションを使用することで、アプリケーション内にある、現在のあるいは別のコントローラの特定のアクションをフォームに指定することができます。
たとえば、 現在のコントローラの ``login()`` アクションをフォームで指定したい場合、 $options 配列を以下のように設定します。::

    echo $this->Form->create($article, ['url' => ['action' => 'login']]);

出力:

.. code-block:: html

    <form method="post" action="/users/login">

もし、現在のコントローラに、予定のフォームアクションがない場合は、フォームアクションの完全なURLを指定することができます。
提供されるURLは、CakePHPアプリケーションに関連します。::

    echo $this->Form->create(null, [
        'url' => ['controller' => 'Articles', 'action' => 'publish']
    ]);

出力:

.. code-block:: html

    <form method="post" action="/articles/publish">

あるいは、外部のドメインを指定できます::

    echo $this->Form->create(null, [
        'url' => 'http://www.google.com/search',
        'type' => 'get'
    ]);

出力:

.. code-block:: html

    <form method="get" action="http://www.google.com/search">

もし、フォームアクションでURLを出力したくない場合は ``'url' => false`` を利用してください。

カスタム張りデーターを使用する
------------------------------

モデルが複数のバリデーションセットを持っていることがしばしばあり、コントローラのアクションが、
適用しようとしている特定のバリデーションルールに基づいて必要となるフィールドをFormHelperが示してほしいと思うことがあるでしょう。
例えば、Usersテーブルが、アカウントが登録されるときにだけ適用される特定のバリデーションルールを守っているとします。::

    echo $this->Form->create($user, [
        'context' => ['validator' => 'register']
    ]);

上述の例は、 ``register`` バリデータを ``$user`` と関連するアソシエーションに対して使う、というものです。
関連付けられたエンティティのフォームを作成しているとき、それぞれのアソシエーションに対して配列を用いることでバリデーションルールを定義できます。::

    echo $this->Form->create($user, [
        'context' => [
            'validator' => [
                'Users' => 'register',
                'Comments' => 'default'
            ]
        ]
    ]);

上述の例は ``register`` をユーザに用い、 ``default`` をユーザのコメントに用います。

コンテキストクラスを作成する
----------------------------

遭遇するであろう基本的なケースをビルトインコンテキストクラスがカバーするつもりであるとき、異なるORMを利用しているなら新しいコンテキストクラスをビルドする必要があるでしょう。
このような状況においては、 `Cake\\View\\Form\\ContextInterface
<http://api.cakephp.org/3.0/class-Cake.View.Form.ContextInterface.html>`_ を実装する必要があります。
``View.beforeRender`` イベントリスナ、あるいはアプリケーションのビュークラスにおいて、これをするのが一番良いこともしばしばあります。::

    $this->Form->addContextProvider('myprovider', function ($request, $data) {
        if ($data['entity'] instanceof MyOrmClass) {
            return new MyProvider($request, $data);
        }
    });

コンテキスト生成機能は、フォームオプションがエンティティの正しい種類かをチェックするロジックを追加できるところです。
入力データを照合するとき、オブジェクトを返していることが分かるでしょう。
もし合致するものがない場合は null を返します。

.. _automagic-form-elements:

フォームの input を作成する
===========================

.. php:method:: input(string $fieldName, array $options = [])

``input()`` メソッドによって、完全なフォームの input を作成することができます。
これらの input は、div, label での囲み、 input ウィジェット、そして必要ならばバリデーションエラーも含んでいます。
フォームのコンテキストでメタデータを使用すると、このメソッドは適切な input タイプを書くフィールドに対して選択します。
内部的には、 ``input()`` は FormHelper の異なるメソッドを使用しています。

作成されたinputのタイプは、カラムのデータタイプに依存します。:

Column Type
    Resulting Form Field
string, uuid (char, varchar, etc.)
    text
boolean, tinyint(1)
    checkbox
decimal
    number
float
    number
integer
    number
text
    textarea
text, with name of password, passwd
    password
text, with name of email
    email
text, with name of tel, telephone, or phone
    tel
date
    day, month, and year selects
datetime, timestamp
    day, month, year, hour, minute, and meridian selects
time
    hour, minute, and meridian selects
binary
    file

もし必要ならば ``$options`` パラメータを用いて、特定のinputタイプを選択できます。::

    echo $this->Form->input('published', ['type' => 'checkbox']);

.. _html5-required:

divで囲う場合は、 もし、モデルのフィールドが必要かつ空であってはいけない、とバリデーションルールが指定していた場合、 ``required`` クラス名が付加されるでしょう。
requiredオプションを用いて、この自動的なフラグ設定を無効にすることができます。::

    echo $this->Form->input('title', ['required' => false]);

全てのフォームで引き起こされるブラウザのバリデーションをスキップするために、
:php:meth:`~Cake\\View\\Helper\\FormHelper::submit()` を用いて ``'formnovalidate' => true`` オプションを作成したinputボタンに対して設定することができます。
あるいは、  :php:meth:`~Cake\\View\\Helper\\FormHelper::create()` に対して ``'novalidate' => true``を設定することができます。

例えば、username (varchar), password (varchar), approved (datetime) そして quote (text) のフィールドを含むUserモデルについて考えてみましょう。
FormHelperの input() メソッドを使って、全てのフォームのフィールドの対して適切なinputを作成できます。::

    echo $this->Form->create($user);
    // Text
    echo $this->Form->input('username');
    // Password
    echo $this->Form->input('password');
    // Day, month, year, hour, minute, meridian
    echo $this->Form->input('approved');
    // Textarea
    echo $this->Form->input('quote');

    echo $this->Form->button('Add');
    echo $this->Form->end();

より広範囲な例をお見せしましょう。日付のフィールドに対するオプションです。::

    echo $this->Form->input('birth_dt', [
        'label' => 'Date of birth',
        'minYear' => date('Y') - 70,
        'maxYear' => date('Y') - 18,
    ]);

下に見られるような ``input()`` の特別なオプションのほかに、inputタイプや ``onfocus`` のようなHTMLアトリビュートのオプションを設定することができます。

belongsTo あるいは hasOne のリレーションを用いているとき、セレクトフィールドを作成したいと思うことがあれば、
User belongsTo Group を仮定すると、以下のものを Users コントローラに追加することができます。::

    $this->set('groups', $this->Users->Groups->find('list'));

そのあとで、ビューテンプレートに以下を追加します。::

    echo $this->Form->input('group_id', ['options' => $groups]);

belongsToManyアソシエーションのGroupsに対してセレクトボックスを作成するために、以下を UsersController に追加できます。::

    $this->set('groups', $this->Users->Groups->find('list'));

そのあとで、ビューテンプレートに以下を追加します。::

    echo $this->Form->input('groups._ids', ['options' => $groups]);

"UserGroup" のようにモデルの名前が2語以上の場合、 set() を用いてデータを渡す際はデータを以下のように複数形かつキャメルケースにして渡します。::

    $this->set('userGroups', $this->UserGroups->find('list'));

.. note::

    送信ボタンを生成するときに ``FormHelper::input()`` を使わないようにしましょう。代わりに
    :php:meth:`~Cake\\View\\Helper\\FormHelper::submit()` を使ってください。

フィールド命名規則
------------------

inputウィジェットを作成しているとき、フォームのエンティティ内の属性と一致するようにフィールドを命名しましょう。
たとえば、 ``$article`` のフォームを作るなら、フィールドは、プロパティにならって作成します。例えば ``title``, ``body`` そして ``published`` というようにです。

アソシエーションをもつモデルおよび任意のモデルのinputは、第1引数として ``association.fieldname`` を渡して作成されます。::

    echo $this->Form->input('association.fieldname');

フィールド名のドットはネストされたリクエストデータに変換されます。
たとえば、 ``0.comments.body`` という名前のフィールドを作成したら、 ``0[comments][body]`` のような名前の属性を得られます。
この規約はORMを用いたデータの保存をやりやすくします。
:ref:`associated-form-inputs` セクションで、さまざまなアソシエーションの種類の詳細を見ることができます。

datetimeに関連するinputを作成するとき、FormHelperはフィールドの接尾辞を付与します。
``year``, ``month``, ``day``, ``hour``, ``minute``, あるいは ``meridian`` が追加された名前がついた追加のフィールドに気づくでしょう。
これらのフィールドはエンティティがマーシャル化されたときに自動的に ``DateTime`` オブジェクトに変換されます。


オプション
----------

``FormHelper::input()`` はたくさんのオプションをサポートしています。
自身のオプションに加えて、 ``input()`` はHTMLアトリビュートと同様、生成されるinputのtypeのオプションも受け入れます。
以下は ``FormHelper::input()`` に特定のオプションをカバーします。

* ``$options['type']`` 内部のモデルを上書きして、typeを指定することで、inputのtypeを強制することができます。
  :ref:`automagic-form-elements` にあるフィールドタイプに加え、 'file', 'password', そしてHTML5でサポートされているあらゆるタイプも作成できます。::

    echo $this->Form->input('field', ['type' => 'file']);
    echo $this->Form->input('email', ['type' => 'email']);

  出力:

  .. code-block:: html

    <div class="input file">
        <label for="field">Field</label>
        <input type="file" name="field" value="" id="field" />
    </div>
    <div class="input email">
        <label for="email">Email</label>
        <input type="email" name="email" value="" id="email" />
    </div>

* ``$options['label']`` 通常inputに伴うラベルとして表示したい文字列をこのキーに設定してください。::

    echo $this->Form->input('name', [
        'label' => 'The User Alias'
    ]);

  出力:

  .. code-block:: html

    <div class="input">
        <label for="name">The User Alias</label>
        <input name="name" type="text" value="" id="name" />
    </div>

  代わりに、ラベルの出力を無効にしたい場合はこのキーを ``false`` に設定してください。::

    echo $this->Form->input('name', ['label' => false]);

  出力:

  .. code-block:: html

    <div class="input">
        <input name="name" type="text" value="" id="name" />
    </div>

  ``label`` エレメントに追加でオプションを指定したい場合は、配列をこれに設定してください。
  こうした場合、ラベルのテキストを編集する場合、この配列の ``text`` キーを用いることができます。::

    echo $this->Form->input('name', [
        'label' => [
            'class' => 'thingy',
            'text' => 'The User Alias'
        ]
    ]);

  出力:

  .. code-block:: html

    <div class="input">
        <label for="name" class="thingy">The User Alias</label>
        <input name="name" type="text" value="" id="name" />
    </div>

* ``$options['error']`` このキーを使うことで、デフォルトのモデルのエラーメッセージを上書きすることができます。
  例えばi18nメッセージを設定する際に用いられます。

  エラーメッセージの出力およびフィールドクラスを無効にしたい場合、このエラーのキーに ``false`` を設定してください。::

    echo $this->Form->input('name', ['error' => false]);

  モデルのエラーメッセージを上書きするためには、もとのバリデーションエラーメッセージと一致するキーを利用して配列を使ってください。::

    $this->Form->input('name', [
        'error' => ['Not long enough' => __('This is not long enough')]
    ]);

  上述の通り、モデルにあるそれぞれのバリデーションルールに対してエラーメッセージを設定できます。
  さらに、i18nメッセージをフォームに与えることもできます。

特定のタイプのinputを作成する
=============================

In addition to the generic ``input()`` method, ``FormHelper`` has specific
methods for generating a number of different types of inputs. These can be used
to generate just the input widget itself, and combined with other methods like
:php:meth:`~Cake\\View\\Helper\\FormHelper::label()` and
:php:meth:`~Cake\\View\\Helper\\FormHelper::error()` to generate fully custom
form layouts.

.. _general-input-options:

Common Options
--------------

Many of the various input element methods support a common set of options. All
of these options are also supported by ``input()``. To reduce repetition the
common options shared by all input methods are as follows:

* ``$options['id']`` Set this key to force the value of the DOM id for the input.
  This will override the idPrefix that may be set.

* ``$options['default']`` Used to set a default value for the input field. The
  value is used if the data passed to the form does not contain a value for the
  field (or if no data is passed at all).

  Example usage::

    echo $this->Form->text('ingredient', ['default' => 'Sugar']);

  Example with select field (Size "Medium" will be selected as
  default)::

    $sizes = ['s' => 'Small', 'm' => 'Medium', 'l' => 'Large'];
    echo $this->Form->select('size', $sizes, ['default' => 'm']);

  .. note::

    You cannot use ``default`` to check a checkbox - instead you might
    set the value in ``$this->request->data`` in your controller,
    or set the input option ``checked`` to ``true``.

    Date and datetime fields' default values can be set by using the
    'selected' key.

    Beware of using ``false`` to assign a default value. A ``false`` value is
    used to disable/exclude options of an input field, so ``'default' => false``
    would not set any value at all. Instead use ``'default' => 0``.

In addition to the above options, you can mixin any HTML attribute you wish to
use. Any non-special option name will be treated as an HTML attribute, and
applied to the generated HTML input element.

Options for Select, Checkbox and Radio Inputs
---------------------------------------------

* ``$options['value']`` Used in combination with a select-type input (i.e.
  For types select, date, time, datetime). Set 'value' to the value of the
  item you wish to be selected by default when the input is rendered::

    echo $this->Form->time('close_time', [
        'value' => '13:30:00'
    ]);

  .. note::

    The value key for date and datetime inputs may also be a UNIX
    timestamp, or a DateTime object.

  For select input where you set the ``multiple`` attribute to true,
  you can use an array of the values you want to select by default::

    echo $this->Form->select('rooms', [
        'multiple' => true,
        // options with values 1 and 3 will be selected as default
        'default' => [1, 3]
    ]);

* ``$options['empty']`` If set to ``true``, forces the input to remain empty.

  When passed to a select list, this creates a blank option with an
  empty value in your drop down list. If you want to have a empty
  value with text displayed instead of just a blank option, pass in a
  string to empty::

      echo $this->Form->select(
          'field',
          [1, 2, 3, 4, 5],
          ['empty' => '(choose one)']
      );

  Output:

  .. code-block:: html

      <select name="field">
          <option value="">(choose one)</option>
          <option value="0">1</option>
          <option value="1">2</option>
          <option value="2">3</option>
          <option value="3">4</option>
          <option value="4">5</option>
      </select>

  Options can also supplied as key-value pairs.

* ``$options['hiddenField']`` For certain input types (checkboxes, radios) a
  hidden input is created so that the key in $this->request->data will exist
  even without a value specified:

  .. code-block:: html

    <input type="hidden" name="published" value="0" />
    <input type="checkbox" name="published" value="1" />

  This can be disabled by setting the ``$options['hiddenField'] = false``::

    echo $this->Form->checkbox('published', ['hiddenField' => false]);

  Which outputs:

  .. code-block:: html

    <input type="checkbox" name="published" value="1">

  If you want to create multiple blocks of inputs on a form that are
  all grouped together, you should use this parameter on all inputs
  except the first. If the hidden input is on the page in multiple
  places, only the last group of input's values will be saved

  In this example, only the tertiary colors would be passed, and the
  primary colors would be overridden:

  .. code-block:: html

    <h2>Primary Colors</h2>
    <input type="hidden" name="color" value="0" />
    <label for="color-red">
        <input type="checkbox" name="color[]" value="5" id="color-red" />
        Red
    </label>

    <label for="color-blue">
        <input type="checkbox" name="color[]" value="5" id="color-blue" />
        Blue
    </label>

    <label for="color-yellow">
        <input type="checkbox" name="color[]" value="5" id="color-yellow" />
        Yellow
    </label>

    <h2>Tertiary Colors</h2>
    <input type="hidden" name="color" value="0" />
    <label for="color-green">
        <input type="checkbox" name="color[]" value="5" id="color-green" />
        Green
    </label>
    <label for="color-purple">
        <input type="checkbox" name="color[]" value="5" id="color-purple" />
        Purple
    </label>
    <label for="color-orange">
        <input type="checkbox" name="color[]" value="5" id="color-orange" />
        Orange
    </label>

  Disabling the ``'hiddenField'`` on the second input group would
  prevent this behavior.

  You can set a different hidden field value other than 0 such as 'N'::

      echo $this->Form->checkbox('published', [
          'value' => 'Y',
          'hiddenField' => 'N',
      ]);

Datetime Options
----------------

* ``$options['timeFormat']`` Used to specify the format of the select inputs for
  a time-related set of inputs. Valid values include ``12``, ``24``, and ``null``.

* ``$options['minYear'], $options['maxYear']`` Used in combination with a
  date/datetime input. Defines the lower and/or upper end of values shown in the
  years select field.

* ``$options['orderYear']`` Used in combination with a date/datetime input.
  Defines the order in which the year values will be set. Valid values include
  'asc', 'desc'. The default value is 'desc'.

* ``$options['interval']`` This option specifies the number of minutes between
  each option in the minutes select box::

    echo $this->Form->input('time', [
        'type' => 'time',
        'interval' => 15
    ]);

  Would create 4 options in the minute select. One for each 15
  minutes.

* ``$options['round']`` Can be set to `up` or `down` to force rounding in either
  direction. Defaults to null which rounds half up according to `interval`.

* ``$options['monthNames']`` If ``false``, 2 digit numbers will be used instead
  of text. If it is given an array like ``['01' => 'Jan', '02' => 'Feb', ...]``
  then the given array will be used.

Creating Input Elements
=======================

Creating Text Inputs
--------------------

.. php:method:: text(string $name, array $options)

The rest of the methods available in the FormHelper are for
creating specific form elements. Many of these methods also make
use of a special $options parameter. In this case, however,
$options is used primarily to specify HTML tag attributes (such as
the value or DOM id of an element in the form)::

    echo $this->Form->text('username', ['class' => 'users']);

Will output:

.. code-block:: html

    <input name="username" type="text" class="users">

Creating Password Inputs
------------------------

.. php:method:: password(string $fieldName, array $options)

Creates a password field. ::

    echo $this->Form->password('password');

Will output:

.. code-block:: html

    <input name="password" value="" type="password">

Creating Hidden Inputs
----------------------

.. php:method:: hidden(string $fieldName, array $options)

Creates a hidden form input. Example::

    echo $this->Form->hidden('id');

Will output:

.. code-block:: html

    <input name="id" value="10" type="hidden" />

Creating Textareas
------------------

.. php:method:: textarea(string $fieldName, array $options)

Creates a textarea input field. ::

    echo $this->Form->textarea('notes');

Will output:

.. code-block:: html

    <textarea name="notes"></textarea>

If the form is edited (that is, the array ``$this->request->data`` will
contain the information saved for the ``User`` model), the value
corresponding to ``notes`` field will automatically be added to the HTML
generated. Example:

.. code-block:: html

    <textarea name="notes" id="notes">
    This text is to be edited.
    </textarea>

.. note::

    The ``textarea`` input type allows for the ``$options`` attribute
    of ``'escape'`` which determines whether or not the contents of the
    textarea should be escaped. Defaults to ``true``.

::

    echo $this->Form->textarea('notes', ['escape' => false]);
    // OR....
    echo $this->Form->input('notes', ['type' => 'textarea', 'escape' => false]);


**Options**

In addition to the :ref:`general-input-options`, textarea() supports a few
specific options:

* ``$options['rows'], $options['cols']`` These two keys specify the number of
  rows and columns::

    echo $this->Form->textarea('textarea', ['rows' => '5', 'cols' => '5']);

  Output:

.. code-block:: html

    <textarea name="textarea" cols="5" rows="5">
    </textarea>

Creating Checkboxes
-------------------

.. php:method:: checkbox(string $fieldName, array $options)

Creates a checkbox form element. This method also generates an
associated hidden form input to force the submission of data for
the specified field. ::

    echo $this->Form->checkbox('done');

Will output:

.. code-block:: html

    <input type="hidden" name="done" value="0">
    <input type="checkbox" name="done" value="1">

It is possible to specify the value of the checkbox by using the
$options array::

    echo $this->Form->checkbox('done', ['value' => 555]);

Will output:

.. code-block:: html

    <input type="hidden" name="done" value="0">
    <input type="checkbox" name="done" value="555">

If you don't want the Form helper to create a hidden input::

    echo $this->Form->checkbox('done', ['hiddenField' => false]);

Will output:

.. code-block:: html

    <input type="checkbox" name="done" value="1">


Creating Radio Buttons
----------------------

.. php:method:: radio(string $fieldName, array $options, array $attributes)

Creates a set of radio button inputs.

**Attributes**

* ``value`` - Indicates the value when this radio button is checked.
* ``label`` - boolean to indicate whether or not labels for widgets should be
  displayed.
* ``hiddenField`` - boolean to indicate if you want the results of radio() to
  include a hidden input with a value of ''. This is useful for creating radio
  sets that are non-continuous.
* ``disabled`` - Set to ``true`` or ``disabled`` to disable all the radio
  buttons.
* ``empty`` - Set to ``true`` to create an input with the value '' as the first
  option. When ``true`` the radio label will be 'empty'. Set this option to
  a string to control the label value.

Generally ``$options`` is a simple key => value pair. However, if you need to
put custom attributes on your radio buttons you can use an expanded format::

    echo $this->Form->radio(
        'favorite_color',
        [
            ['value' => 'r', 'text' => 'Red', 'style' => 'color:red;'],
            ['value' => 'u', 'text' => 'Blue', 'style' => 'color:blue;'],
            ['value' => 'g', 'text' => 'Green', 'style' => 'color:green;'],
        ]
    );

    // Will output
    <input type="hidden" name="favorite_color" value="">
    <label for="favorite-color-r">
        <input type="radio" name="favorite_color" value="r" style="color:red;" id="favorite-color-r">
        Red
    </label>
    <label for="favorite-color-u">
        <input type="radio" name="favorite_color" value="u" style="color:blue;" id="favorite-color-u">
        Blue
    </label>
    <label for="favorite-color-g">
        <input type="radio" name="favorite_color" value="g" style="color:green;" id="favorite-color-g">
        Green
    </label>

Creating Select Pickers
-----------------------

.. php:method:: select(string $fieldName, array $options, array $attributes)

Creates a select element, populated with the items in ``$options``,
with the option specified by ``$attributes['value']`` shown as selected by
default. Set the 'empty' key in the ``$attributes`` variable to ``false`` to
turn off the default empty option::

    $options = ['M' => 'Male', 'F' => 'Female'];
    echo $this->Form->select('gender', $options);

Will output:

.. code-block:: html

    <select name="gender">
    <option value=""></option>
    <option value="M">Male</option>
    <option value="F">Female</option>
    </select>

The ``select`` input type allows for a special ``$option``
attribute called ``'escape'`` which accepts a bool and determines
whether to HTML entity encode the contents of the select options.
Defaults to ``true``::

    $options = ['M' => 'Male', 'F' => 'Female'];
    echo $this->Form->select('gender', $options, ['escape' => false]);

* ``$attributes['options']`` This key allows you to manually specify options for
  a select input, or for a radio group. Unless the 'type' is specified as
  'radio', the FormHelper will assume that the target output is a select input::

    echo $this->Form->select('field', [1,2,3,4,5]);

  Output:

  .. code-block:: html

    <select name="field">
        <option value="0">1</option>
        <option value="1">2</option>
        <option value="2">3</option>
        <option value="3">4</option>
        <option value="4">5</option>
    </select>

  Options can also be supplied as key-value pairs::

    echo $this->Form->select('field', [
        'Value 1' => 'Label 1',
        'Value 2' => 'Label 2',
        'Value 3' => 'Label 3'
    ]);

  Output:

  .. code-block:: html

    <select name="field">
        <option value="Value 1">Label 1</option>
        <option value="Value 2">Label 2</option>
        <option value="Value 3">Label 3</option>
    </select>

  If you would like to generate a select with optgroups, just pass
  data in hierarchical format. This works on multiple checkboxes and radio
  buttons too, but instead of optgroups wraps elements in fieldsets::

    $options = [
       'Group 1' => [
          'Value 1' => 'Label 1',
          'Value 2' => 'Label 2'
       ],
       'Group 2' => [
          'Value 3' => 'Label 3'
       ]
    ];
    echo $this->Form->select('field', $options);

  Output:

  .. code-block:: html

    <select name="field">
        <optgroup label="Group 1">
            <option value="Value 1">Label 1</option>
            <option value="Value 2">Label 2</option>
        </optgroup>
        <optgroup label="Group 2">
            <option value="Value 3">Label 3</option>
        </optgroup>
    </select>

To generate attributes within an option tag::

    $options = [
        [ 'text' => 'Description 1', 'value' => 'value 1', 'attr_name' => 'attr_value 1' ],
        [ 'text' => 'Description 2', 'value' => 'value 2', 'attr_name' => 'attr_value 2' ],
        [ 'text' => 'Description 3', 'value' => 'value 3', 'other_attr_name' => 'other_attr_value' ],
    ];
    echo $this->Form->select('field', $options);

Output:

.. code-block:: html

    <select name="field">
        <option value="value 1" attr_name="attr_value 1">Description 1</option>
        <option value="value 2" attr_name="attr_value 2">Description 2</option>
        <option value="value 3" other_attr_name="other_attr_value">Description 3</option>
    </select>

* ``$attributes['multiple']`` If 'multiple' has been set to ``true`` for an
  input that outputs a select, the select will allow multiple selections::

    echo $this->Form->select('field', $options, ['multiple' => true]);

  Alternatively set 'multiple' to 'checkbox' to output a list of
  related check boxes::

    $options = [
        'Value 1' => 'Label 1',
        'Value 2' => 'Label 2'
    ];
    echo $this->Form->select('field', $options, [
        'multiple' => 'checkbox'
    ]);

  Output:

  .. code-block:: html

      <input name="field" value="" type="hidden">
      <div class="checkbox">
        <label for="field-1">
         <input name="field[]" value="Value 1" id="field-1" type="checkbox">
         Label 1
         </label>
      </div>
      <div class="checkbox">
         <label for="field-2">
         <input name="field[]" value="Value 2" id="field-2" type="checkbox">
         Label 2
         </label>
      </div>

* ``$attributes['disabled']`` When creating checkboxes, this option can be set
  to disable all or some checkboxes. To disable all checkboxes set disabled
  to ``true``::

    $options = [
        'Value 1' => 'Label 1',
        'Value 2' => 'Label 2'
    ];
    echo $this->Form->select('field', $options, [
        'multiple' => 'checkbox',
        'disabled' => ['Value 1']
    ]);

  Output:

  .. code-block:: html

       <input name="field" value="" type="hidden">
       <div class="checkbox">
          <label for="field-1">
          <input name="field[]" disabled="disabled" value="Value 1" type="checkbox">
          Label 1
          </label>
       </div>
       <div class="checkbox">
          <label for="field-2">
          <input name="field[]" value="Value 2" id="field-2" type="checkbox">
          Label 2
          </label>
       </div>

Creating File Inputs
--------------------

.. php:method:: file(string $fieldName, array $options)

To add a file upload field to a form, you must first make sure that
the form enctype is set to "multipart/form-data", so start off with
a create function such as the following::

    echo $this->Form->create($document, ['enctype' => 'multipart/form-data']);
    // OR
    echo $this->Form->create($document, ['type' => 'file']);

Next add either of the two lines to your form view file::

    echo $this->Form->input('submittedfile', [
        'type' => 'file'
    ]);

    // OR
    echo $this->Form->file('submittedfile');

Due to the limitations of HTML itself, it is not possible to put
default values into input fields of type 'file'. Each time the form
is displayed, the value inside will be empty.

Upon submission, file fields provide an expanded data array to the
script receiving the form data.

For the example above, the values in the submitted data array would
be organized as follows, if the CakePHP was installed on a Windows
server. 'tmp\_name' will have a different path in a Unix
environment::

    $this->request->data['submittedfile'] = [
        'name' => 'conference_schedule.pdf',
        'type' => 'application/pdf',
        'tmp_name' => 'C:/WINDOWS/TEMP/php1EE.tmp',
        'error' => 0, // On Windows this can be a string.
        'size' => 41737,
    ];

This array is generated by PHP itself, so for more detail on the
way PHP handles data passed via file fields
`read the PHP manual section on file uploads <http://php.net/features.file-upload>`_.

.. note::

    When using ``$this->Form->file()``, remember to set the form
    encoding-type, by setting the type option to 'file' in
    ``$this->Form->create()``.

Creating DateTime Inputs
------------------------

.. php:method:: dateTime($fieldName, $options = [])

Creates a set of select inputs for date and time. This method accepts a number
of options:

* ``monthNames`` If ``false``, 2 digit numbers will be used instead of text.
  If an array, the given array will be used.
* ``minYear`` The lowest year to use in the year select
* ``maxYear`` The maximum year to use in the year select
* ``interval`` The interval for the minutes select. Defaults to 1
* ``empty`` - If ``true``, the empty select option is shown. If a string,
  that string is displayed as the empty element.
* ``round`` - Set to ``up`` or ``down`` if you want to force rounding in either
  direction. Defaults to null.
* ``default`` The default value to be used by the input. A value in
  ``$this->request->data`` matching the field name will override this value. If
  no default is provided ``time()`` will be used.
* ``timeFormat`` The time format to use, either 12 or 24.
* ``second`` Set to ``true`` to enable seconds drop down.

To control the order of inputs, and any elements/content between the inputs you
can override the ``dateWidget`` template. By default the ``dateWidget`` template
is::

    {{year}}{{month}}{{day}}{{hour}}{{minute}}{{second}}{{meridian}}

To create a datetime inputs with custom classes/attributes on a specific select
box, you can use the options in each component::

    echo $this->Form->datetime('released', [
        'year' => [
            'class' => 'year-classname',
        ],
        'month' => [
            'class' => 'month-class',
            'data-type' => 'month',
        ],
    ]);

Which would create the following two selects:

.. code-block:: html

    <select name="released[year]" class="year-class">
        <option value="" selected="selected"></option>
        <option value="00">0</option>
        <option value="01">1</option>
        <!-- .. snipped for brevity .. -->
    </select>
    <select name="released[month]" class="month-class" data-type="month">
        <option value="" selected="selected"></option>
        <option value="01">January</option>
        <!-- .. snipped for brevity .. -->
    </select>

Creating Time Inputs
--------------------

.. php:method:: time($fieldName, $options = [])

Creates two select elements populated with 24 hours and 60 minutes for ``hour``
and ``minute``, respectively.
Additionally, HTML attributes may be supplied in $options for each specific
``type``. If ``$options['empty']`` is ``false``, the select will not include an
empty option:

* ``empty`` - If ``true``, the empty select option is shown. If a string,
  that string is displayed as the empty element.
* ``default`` | ``value`` The default value to be used by the input. A value in
  ``$this->request->data`` matching the field name will override this value.
  If no default is provided ``time()`` will be used.
* ``timeFormat`` The time format to use, either 12 or 24. Defaults to 24.
* ``second`` Set to ``true`` to enable seconds drop down.
* ``interval`` The interval for the minutes select. Defaults to 1.

For example, to create a time range with minutes selectable in 15 minute
increments, and to apply classes to the select boxes, you could do the
following::

    echo $this->Form->time('released', [
        'interval' => 15,
        'hour' => [
            'class' => 'foo-class',
        ],
        'minute' => [
            'class' => 'bar-class',
        ],
    ]);

Which would create the following two selects:

.. code-block:: html

    <select name="released[hour]" class="foo-class">
        <option value="" selected="selected"></option>
        <option value="00">0</option>
        <option value="01">1</option>
        <!-- .. snipped for brevity .. -->
        <option value="22">22</option>
        <option value="23">23</option>
    </select>
    <select name="released[minute]" class="bar-class">
        <option value="" selected="selected"></option>
        <option value="00">00</option>
        <option value="15">15</option>
        <option value="30">30</option>
        <option value="45">45</option>
    </select>

Creating Year Inputs
--------------------

.. php:method:: year(string $fieldName, array $options = [])

Creates a select element populated with the years from ``minYear``
to ``maxYear``. Additionally, HTML attributes may be supplied in $options. If
``$options['empty']`` is ``false``, the select will not include an
empty option:

* ``empty`` - If ``true``, the empty select option is shown. If a string,
  that string is displayed as the empty element.
* ``orderYear`` - Ordering of year values in select options.
  Possible values 'asc', 'desc'. Default 'desc'
* ``value`` The selected value of the input.
* ``maxYear`` The max year to appear in the select element.
* ``minYear`` The min year to appear in the select element.

For example, to create a year range from 2000 to the current year you
would do the following::

    echo $this->Form->year('purchased', [
        'minYear' => 2000,
        'maxYear' => date('Y')
    ]);

If it was 2009, you would get the following:

.. code-block:: html

    <select name="purchased[year]">
    <option value=""></option>
    <option value="2009">2009</option>
    <option value="2008">2008</option>
    <option value="2007">2007</option>
    <option value="2006">2006</option>
    <option value="2005">2005</option>
    <option value="2004">2004</option>
    <option value="2003">2003</option>
    <option value="2002">2002</option>
    <option value="2001">2001</option>
    <option value="2000">2000</option>
    </select>

Creating Month Inputs
---------------------

.. php:method:: month(string $fieldName, array $attributes)

Creates a select element populated with month names::

    echo $this->Form->month('mob');

Will output:

.. code-block:: html

    <select name="mob[month]">
    <option value=""></option>
    <option value="01">January</option>
    <option value="02">February</option>
    <option value="03">March</option>
    <option value="04">April</option>
    <option value="05">May</option>
    <option value="06">June</option>
    <option value="07">July</option>
    <option value="08">August</option>
    <option value="09">September</option>
    <option value="10">October</option>
    <option value="11">November</option>
    <option value="12">December</option>
    </select>

You can pass in your own array of months to be used by setting the
'monthNames' attribute, or have months displayed as numbers by
passing ``false``. (Note: the default months can be localized with CakePHP
:doc:`/core-libraries/internationalization-and-localization` features.)::

    echo $this->Form->month('mob', ['monthNames' => false]);

Creating Day Inputs
-------------------

.. php:method:: day(string $fieldName, array $attributes)

Creates a select element populated with the (numerical) days of the
month.

To create an empty option with prompt text of your choosing (e.g.
the first option is 'Day'), you can supply the text as the final
parameter as follows::

    echo $this->Form->day('created');

Will output:

.. code-block:: html

    <select name="created[day]">
    <option value=""></option>
    <option value="01">1</option>
    <option value="02">2</option>
    <option value="03">3</option>
    ...
    <option value="31">31</option>
    </select>

Creating Hour Inputs
--------------------

.. php:method:: hour(string $fieldName, array $attributes)

Creates a select element populated with the hours of the day. You can
create either 12 or 24 hour pickers using the format option::

    echo $this->Form->hour('created', [
        'format' => 12
    ]);
    echo $this->Form->hour('created', [
        'format' => 24
    ]);

Creating Minute Inputs
----------------------

.. php:method:: minute(string $fieldName, array $attributes)

Creates a select element populated with the minutes of the hour. You
can create a select that only contains specific values using the ``interval``
option. For example, if you wanted 10 minute increments you would do the
following::

    echo $this->Form->minute('created', [
        'interval' => 10
    ]);

Creating Meridian Inputs
------------------------

.. php:method:: meridian(string $fieldName, array $attributes)

Creates a select element populated with 'am' and 'pm'.

Creating Labels
===============

.. php:method:: label(string $fieldName, string $text, array $options)

Create a label element. ``$fieldName`` is used for generating the
DOM id. If ``$text`` is undefined, ``$fieldName`` will be used to inflect
the label's text::

    echo $this->Form->label('User.name');
    echo $this->Form->label('User.name', 'Your username');

Output:

.. code-block:: html

    <label for="user-name">Name</label>
    <label for="user-name">Your username</label>

``$options`` can either be an array of HTML attributes, or a string that
will be used as a class name::

    echo $this->Form->label('User.name', null, ['id' => 'user-label']);
    echo $this->Form->label('User.name', 'Your username', 'highlight');

Output:

.. code-block:: html

    <label for="user-name" id="user-label">Name</label>
    <label for="user-name" class="highlight">Your username</label>

Displaying and Checking Errors
==============================

.. php:method:: error(string $fieldName, mixed $text, array $options)

Shows a validation error message, specified by $text, for the given
field, in the event that a validation error has occurred.

Options:

-  'escape' bool Whether or not to HTML escape the contents of the
   error.

.. TODO:: Add examples.

.. php:method:: isFieldError(string $fieldName)

Returns ``true`` if the supplied $fieldName has an active validation
error. ::

    if ($this->Form->isFieldError('gender')) {
        echo $this->Form->error('gender');
    }

.. note::

    When using :php:meth:`~Cake\\View\\Helper\\FormHelper::input()`, errors are
    rendered by default.

Creating Buttons and Submit Elements
====================================

.. php:method:: submit(string $caption, array $options)

Creates a submit input with ``$caption`` as the text. If the supplied
``$caption`` is a URL to an image, an image submit button will be generated.
The following::

    echo $this->Form->submit();

Will output:

.. code-block:: html

    <div class="submit"><input value="Submit" type="submit"></div>

You can pass a relative or absolute URL to an image for the
caption parameter instead of caption text::

    echo $this->Form->submit('ok.png');

Will output:

.. code-block:: html

    <div class="submit"><input type="image" src="/img/ok.png"></div>

Submit inputs are useful when you only need basic text or images. If you need
more complex button content you should use ``button()``.

Creating Button Elements
------------------------

.. php:method:: button(string $title, array $options = [])

Creates an HTML button with the specified title and a default type
of "button". Setting ``$options['type']`` will output one of the
three possible button types:

#. submit: Same as the ``$this->Form->submit`` method - (the
   default).
#. reset: Creates a form reset button.
#. button: Creates a standard push button.

::

    echo $this->Form->button('A Button');
    echo $this->Form->button('Another Button', ['type' => 'button']);
    echo $this->Form->button('Reset the Form', ['type' => 'reset']);
    echo $this->Form->button('Submit Form', ['type' => 'submit']);

Will output:

.. code-block:: html

    <button type="submit">A Button</button>
    <button type="button">Another Button</button>
    <button type="reset">Reset the Form</button>
    <button type="submit">Submit Form</button>

The ``button`` input type supports the ``escape`` option, which accepts
a boolean and defaults to ``false``. It determines whether to HTML encode the
``$title`` of the button::

    // Will render escaped HTML.
    echo $this->Form->button('<em>Submit Form</em>', [
        'type' => 'submit',
        'escape' => true
    ]);

Closing the Form
================

.. php:method:: end($secureAttributes = [])

The ``end()`` method closes and completes a form. Often, ``end()`` will only
output a closing form tag, but using ``end()`` is a good practice as it
enables FormHelper to insert hidden form elements that
:php:class:`Cake\\Controller\\Component\\SecurityComponent` requires:

.. code-block:: php

    <?= $this->Form->create(); ?>

    <!-- Form elements go here -->

    <?= $this->Form->end(); ?>

The ``$secureAttributes`` parameter allows you to pass additional HTML
attributes to the hidden inputs that are generated when your application is
using ``SecurityComponent``. If you need to add additional attributes to the
generated hidden inputs you can use the ``$secureAttributes`` argument::

    echo $this->Form->end(['data-type' => 'hidden']);

Will output:

.. code-block:: html

    <div style="display:none;">
        <input type="hidden" name="_Token[fields]" data-type="hidden"
            value="2981c38990f3f6ba935e6561dc77277966fabd6d%3AAddresses.id">
        <input type="hidden" name="_Token[unlocked]" data-type="hidden"
            value="address%7Cfirst_name">
    </div>

.. note::

    If you are using
    :php:class:`Cake\\Controller\\Component\\SecurityComponent` in your
    application you should always end your forms with ``end()``.

Creating Standalone Buttons and POST links
==========================================

.. php:method:: postButton(string $title, mixed $url, array $options = [])

    Create a ``<button>`` tag with a surrounding ``<form>`` that submits via
    POST.

    This method creates a ``<form>`` element. So do not use this method in some
    opened form. Instead use
    :php:meth:`Cake\\View\\Helper\\FormHelper::submit()`
    or :php:meth:`Cake\\View\\Helper\\FormHelper::button()` to create buttons
    inside opened forms.

.. php:method:: postLink(string $title, mixed $url = null, array $options = [])

    Creates an HTML link, but accesses the URL using method POST. Requires
    JavaScript to be enabled in browser.

    This method creates a ``<form>`` element. If you want to use this method
    inside of an existing form, you must use the ``block`` option so that the
    new form is being set to a :ref:`view block <view-blocks>` that can be
    rendered outside of the main form.

    If all you are looking for is a button to submit your form, then you should
    use :php:meth:`Cake\\View\\Helper\\FormHelper::button()` or
    :php:meth:`Cake\\View\\Helper\\FormHelper::submit()` instead.


Customizing the Templates FormHelper Uses
=========================================

Like many helpers in CakePHP, FormHelper uses string templates to format the
HTML it creates. While the default templates are intended to be a reasonable set
of defaults. You may need to customize the templates to suit your application.

To change the templates when the helper is loaded you can set the ``templates``
option when including the helper in your controller::

    // In a View class
    $this->loadHelper('Form', [
        'templates' => 'app_form',
    ]);

This would load the tags in **config/app_form.php**. This file should
contain an array of templates indexed by name::

    // in config/app_form.php
    return [
        'inputContainer' => '<div class="form-control">{{content}}</div>',
    ];

Any templates you define will replace the default ones included in the helper.
Templates that are not replaced, will continue to use the default values.
You can also change the templates at runtime using the ``templates()`` method::

    $myTemplates = [
        'inputContainer' => '<div class="form-control">{{content}}</div>',
    ];
    $this->Form->templates($myTemplates);

.. warning::

    Template strings containing a percentage sign (``%``) need special attention,
    you should prefix this character with another percentage so it looks like
    ``%%``. The reason is that internally templates are compiled to be used with
    ``sprintf()``. Example: '<div style="width:{{size}}%%">{{content}}</div>'

List of Templates
-----------------

The list of default templates, their default format and the variables they
expect can be found at the
`FormHelper API documentation <http://api.cakephp.org/3.2/class-Cake.View.Helper.FormHelper.html#%24_defaultConfig>`_.

In addition to these templates, the ``input()`` method will attempt to use
distinct templates for each input container. For example, when creating
a datetime input the ``datetimeContainer`` will be used if it is present.
If that container is missing the ``inputContainer`` template will be used. For
example::

    // Add custom radio wrapping HTML
    $this->Form->templates([
        'radioContainer' => '<div class="form-radio">{{content}}</div>'
    ]);

    // Create a radio set with our custom wrapping div.
    echo $this->Form->radio('User.email_notifications', ['y', 'n']);

Similar to input containers, the ``input()`` method will also attempt to use
distinct templates for each form group. A form group is a combo of label and
input. For example, when creating a radio input the ``radioFormGroup`` will be
used if it is present. If that template is missing by default each set of label
& input is rendered using the ``formGroup`` template. For example::

    // Add custom radio form group
    $this->Form->templates([
        'radioFormGroup' => '<div class="radio">{{label}}{{input}}</div>'
    ]);

Adding Additional Template Variables to Templates
-------------------------------------------------

You can add additional template placeholders in custom templates, and populate
those placeholders when generating inputs::

    // Add a template with the help placeholder.
    $this->Form->templates([
        'inputContainer' => '<div class="input {{type}}{{required}}">
            {{content}} <span class="help">{{help}}</span></div>'
    ]);

    // Generate an input and populate the help variable
    echo $this->Form->input('password', [
        'templateVars' => ['help' => 'At least 8 characters long.']
    ]);

.. versionadded:: 3.1
    The templateVars option was added in 3.1.0

Moving Checkboxes & Radios Outside of a Label
---------------------------------------------

By default CakePHP nests checkboxes and radio buttons within label elements.
This helps make it easier to integrate popular CSS frameworks. If you need to
place checkbox/radio inputs outside of the label you can do so by modifying the
templates::

    $this->Form->templates([
        'nestingLabel' => '{{input}}<label{{attrs}}>{{text}}</label>',
        'formGroup' => '{{input}}{{label}}',
    ]);

This will make radio buttons and checkboxes render outside of their labels.

Generating Entire Forms
=======================

.. php:method:: inputs(array $fields = [], $options = [])

Generates a set of inputs for the given context wrapped in a fieldset. You can
specify the generated fields by including them::

    echo $this->Form->inputs([
        'name' => ['label' => 'custom label']
    ]);

You can customize the legend text using an option::

    echo $this->Form->inputs($fields, ['legend' => 'Update news post']);

You can customize the generated inputs by defining additional options in the
``$fields`` parameter::

    echo $this->Form->inputs([
        'name' => ['label' => 'custom label']
    ]);

When customizing, ``fields``, you can use the ``$options`` parameter to
control the generated legend/fieldset.

- ``fieldset`` Set to ``false`` to disable the fieldset. You can also pass an
  array of parameters to be applied as HTML attributes to the fieldset tag. If
  you pass an empty array, the fieldset will be displayed without attributes.
- ``legend`` Set to ``false`` to disable the legend for the generated input set.
  Or supply a string to customize the legend text.

For example::

    echo $this->Form->allInputs(
        [
            'name' => ['label' => 'custom label']
        ],
        null,
        ['legend' => 'Update your post']
    );

If you disable the fieldset, the legend will not print.

.. php:method:: allInputs(array $fields, $options = [])

This method is closely related to ``inputs()``, however the ``$fields`` argument
is defaulted to *all* fields in the current top-level entity. To exclude
specific fields from the generated inputs, set them to ``false`` in the fields
parameter::

    echo $this->Form->allInputs(['password' => false]);

.. _associated-form-inputs:

Creating Inputs for Associated Data
===================================

Creating forms for associated data is straightforward and is closely related to
the paths in your entity's data. Assuming the following table relations:

* Authors HasOne Profiles
* Authors HasMany Articles
* Articles HasMany Comments
* Articles BelongsTo Authors
* Articles BelongsToMany Tags

If we were editing an article with its associations loaded we could
create the following inputs::

    $this->Form->create($article);

    // Article inputs.
    echo $this->Form->input('title');

    // Author inputs (belongsTo)
    echo $this->Form->input('author.id');
    echo $this->Form->input('author.first_name');
    echo $this->Form->input('author.last_name');

    // Author profile (belongsTo + hasOne)
    echo $this->Form->input('author.profile.id');
    echo $this->Form->input('author.profile.username');

    // Tags inputs (belongsToMany)
    echo $this->Form->input('tags.0.id');
    echo $this->Form->input('tags.0.name');
    echo $this->Form->input('tags.1.id');
    echo $this->Form->input('tags.1.name');

    // Multiple select element for belongsToMany
    echo $this->Form->input('tags._ids', [
        'type' => 'select',
        'multiple' => true,
        'options' => $tagList,
    ]);

    // Inputs for the joint table (articles_tags)
    echo $this->Form->input('tags.0._joinData.starred');
    echo $this->Form->input('tags.1._joinData.starred');

    // Comments inputs (hasMany)
    echo $this->Form->input('comments.0.id');
    echo $this->Form->input('comments.0.comment');
    echo $this->Form->input('comments.1.id');
    echo $this->Form->input('comments.1.comment');

The above inputs could then be marshalled into a completed entity graph using
the following code in your controller::

    $article = $this->Articles->patchEntity($article, $this->request->data, [
        'associated' => [
            'Authors',
            'Authors.Profiles',
            'Tags',
            'Comments'
        ]
    ]);

Adding Custom Widgets
=====================

CakePHP makes it easy to add custom input widgets in your application, and use
them like any other input type. All of the core input types are implemented as
widgets, which means you can override any core widget with your own
implemenation as well.

Building a Widget Class
-----------------------

Widget classes have a very simple required interface. They must implement the
:php:class:`Cake\\View\\Widget\\WidgetInterface`. This interface requires
the ``render(array $data)`` and ``secureFields(array $data)`` methods to be
implemented. The ``render()`` method expects an array of data to build the
widget and is expected to return a string of HTML for the widget.
The ``secureFields()`` method expects an array of data as well and is expected
to return an array containing the list of fields to secure for this widget.
If CakePHP is constructing your widget you can expect to
get a ``Cake\View\StringTemplate`` instance as the first argument, followed by
any dependencies you define. If we wanted to build an Autocomplete widget you
could do the following::

    namespace App\View\Widget;

    use Cake\View\Form\ContextInterface;
    use Cake\View\Widget\WidgetInterface;

    class AutocompleteWidget implements WidgetInterface
    {

        protected $_templates;

        public function __construct($templates)
        {
            $this->_templates = $templates;
        }

        public function render(array $data, ContextInterface $context)
        {
            $data += [
                'name' => '',
            ];
            return $this->_templates->format('autocomplete', [
                'name' => $data['name'],
                'attrs' => $this->_templates->formatAttributes($data, ['name'])
            ]);
        }

        public function secureFields(array $data)
        {
            return [$data['name']];
        }
    }

Obviously, this is a very simple example, but it demonstrates how a custom
widget could be built.

Using Widgets
-------------

You can load custom widgets when loading FormHelper or by using the
``addWidget()`` method. When loading FormHelper, widgets are defined as
a setting::

    // In View class
    $this->loadHelper('Form', [
        'widgets' => [
            'autocomplete' => ['Autocomplete']
        ]
    ]);

If your widget requires other widgets, you can have FormHelper populate those
dependencies by declaring them::

    $this->loadHelper('Form', [
        'widgets' => [
            'autocomplete' => [
                'App\View\Widget\AutocompleteWidget',
                'text',
                'label'
            ]
        ]
    ]);

In the above example, the autocomplete widget would depend on the ``text`` and
``label`` widgets. If your widget needs access to the View, you should use the
``_view`` 'widget'.  When the autocomplete widget is created, it will be passed
the widget objects that are related to the ``text`` and ``label`` names. To add
widgets using the ``addWidget()`` method would look like::

    // Using a classname.
    $this->Form->addWidget(
        'autocomplete',
        ['Autocomplete', 'text', 'label']
    );

    // Using an instance - requires you to resolve dependencies.
    $autocomplete = new AutocompleteWidget(
        $this->Form->getTemplater(),
        $this->Form->widgetRegistry()->get('text'),
        $this->Form->widgetRegistry()->get('label'),
    );
    $this->Form->addWidget('autocomplete', $autocomplete);

Once added/replaced, widgets can be used as the input 'type'::

    echo $this->Form->input('search', ['type' => 'autocomplete']);

This will create the custom widget with a label and wrapping div just like
``input()`` always does. Alternatively, you can create just the input widget
using the magic method::

    echo $this->Form->autocomplete('search', $options);

Working with SecurityComponent
==============================

:php:meth:`Cake\\Controller\\Component\\SecurityComponent` offers several
features that make your forms safer and more secure. By simply including the
``SecurityComponent`` in your controller, you'll automatically benefit from
form tampering features.

As mentioned previously when using SecurityComponent, you should always close
your forms using :php:meth:`~Cake\\View\\Helper\\FormHelper::end()`. This will
ensure that the special ``_Token`` inputs are generated.

.. php:method:: unlockField($name)

    Unlocks a field making it exempt from the ``SecurityComponent`` field
    hashing. This also allows the fields to be manipulated by JavaScript.
    The ``$name`` parameter should be the entity property name for the input::

        $this->Form->unlockField('id');

.. php:method:: secure(array $fields = [])

    Generates a hidden field with a security hash based on the fields used
    in the form.


.. meta::
    :title lang=en: FormHelper
    :description lang=en: The FormHelper focuses on creating forms quickly, in a way that will streamline validation, re-population and layout.
    :keywords lang=en: html helper,cakephp html,form create,form input,form select,form file field,form label,form text,form password,form checkbox,form radio,form submit,form date time,form error,validate upload,unlock field,form security
