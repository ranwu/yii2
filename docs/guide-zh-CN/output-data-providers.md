数据提供者
==============

In the [Pagination](output-pagination.md) and [Sorting](output-sorting.md) sections, we have described how to
allow end users to choose a particular page of data to display and sort them by some columns. Because the task
of paginating and sorting data is very common, Yii provides a set of *data provider* classes to encapsulate it.

在[分页](output-pagination.md)和[排序](output-sorting.md)章节里，我们讲解了如何让终端用户去选择某数据的特殊页面以及通过一些字段来让它们排序。由于数据分页和排序是非常普遍的任务，所以 Yii 提供了一套 *data provider* 类来封装它。

A data provider is a class implementing [[yii\data\DataProviderInterface]]. It mainly supports retrieving paginated
and sorted data. It is usually used to work with [data widgets](output-data-widgets.md) so that end users can 
interactively paginate and sort data.

数据提供者（data provider）是一个实现 [[yii\data\DataProviderInterface]] 接口的类，主要支持分页检索和数据排序，常常与 [data widgets](output-data-widgets.md) 一起运行以达到终端用户可以交互式地进行分页和排序数据的目的。

The following data provider classes are included in the Yii releases:

以下是包含在 Yii 发行中的数据提供者（data provider）类：

* [[yii\data\ActiveDataProvider]]: uses [[yii\db\Query]] or [[yii\db\ActiveQuery]] to query data from databases
  and return them in terms of arrays or [Active Record](db-active-record.md) instances.
* [[yii\data\ActiveDataProvider]]：使用 [[yii\db\Query]] 或 [[yii\db\ActiveQuery]] 来查询数据库并且以数组或 [Active Record](db-active-record.md) 实例来返回。
* [[yii\data\SqlDataProvider]]: executes a SQL statement and returns database data as arrays.
* [[yii\data\SqlDataProvider]]：执行 SQL 语句和返回以数组形式的数据库数据。
* [[yii\data\ArrayDataProvider]]: takes a big array and returns a slice of it based on the paginating and sorting
  specifications.
* [[yii\data\ArrayDataProvider]]：获得一个大数组和返回一部分基于分页和排序的详细说明

The usage of all these data providers share the following common pattern:

所有这些数据提供者的使用方式分为以下常见模式：

```php
// create the data provider by configuring its pagination and sort properties
// 通过配置它的分页和排序属性来创建数据提供者
$provider = new XyzDataProvider([
    'pagination' => [...],
    'sort' => [...],
]);

// retrieves paginated and sorted data
// 检索分页和排序数据
$models = $provider->getModels();

// get the number of data items in the current page
// 得到当前页面数据项的数目
$count = $provider->getCount();

// get the total number of data items across all pages
// 得到所有页面数据项的总数目
$totalCount = $provider->getTotalCount();
```

You specify the pagination and sorting behaviors of a data provider by configuring its 
[[yii\data\BaseDataProvider::pagination|pagination]] and [[yii\data\BaseDataProvider::sort|sort]] properties
which correspond to the configurations for [[yii\data\Pagination]] and [[yii\data\Sort]], respectively.
You may also configure them to be `false` to disable pagination and/or sorting features.

你可以配置 [[yii\data\BaseDataProvider::pagination|pagination]] 和 [[yii\data\BaseDataProvider::sort|sort]] 属性来规定数据提供者的分页和排序行为，同时它们也分别对应到 [[yii\data\Pagination]] 和 [[yii\data\Sort]] 的配置。你还可以把它们配置为 `false` 来同时（或单独）关闭分页和排序功能。 

[Data widgets](output-data-widgets.md), such as [[yii\grid\GridView]], have a property named `dataProvider` which
can take a data provider instance and display the data it provides. For example,

数据小部件 [Data widgets](output-data-widgets.md)，比如 [[yii\grid\GridView]]，有一个称为 `dataProvider` 的属性，它可以获取数据提供者实例以及显示它提供的数据。比如，

```php
echo yii\grid\GridView::widget([
    'dataProvider' => $dataProvider,
]);
```

These data providers mainly vary in the way how the data source is specified. In the following subsections,
we will explain the detailed usage of each of these data providers.

这些数据提供者主要以源数据的定义方式而变化。在以下子章节中，我们会说明每个数据提供者的详细使用方法。

## Active Data Provider <span id="active-data-provider"></span> 
## 活动数据提供者 <span id="active-data-provider"></span> 

To use [[yii\data\ActiveDataProvider]], you should configure its [[yii\data\ActiveDataProvider::query|query]] property.
It can take either a [[yii\db\Query]] or [[yii\db\ActiveQuery]] object. If the former, the data returned will be arrays;
if the latter, the data returned can be either arrays or [Active Record](db-active-record.md) instances.
For example,

使用 [[yii\data\ActiveDataProvider]]，你应该配置它的 [[yii\data\ActiveDataProvider::query|query]] 属性。他可以得到一个 [[yii\db\Query]] 或 [[yii\db\ActiveQuery]] 对象。如果是前者，数据将会以数组的方式返回；如果是后者，数据可以以数组或活动记录实例 [Active Record](db-active-record.md) 的方式返回。比如，

```php
use yii\data\ActiveDataProvider;

$query = Post::find()->where(['status' => 1]);

$provider = new ActiveDataProvider([
    'query' => $query,
    'pagination' => [
        'pageSize' => 10,
    ],
    'sort' => [
        'defaultOrder' => [
            'created_at' => SORT_DESC,
            'title' => SORT_ASC, 
        ]
    ],
]);

// returns an array of Post objects
// 返回 Post 对象的数组
$posts = $provider->getModels();
```

If `$query` in the above example is created using the following code, then the data provider will return raw arrays.

如果以上例子的 `$query` 变量是通过使用以下代码创建的话，那么这个数据提供者会以原始数组的方式返回。 

```php
use yii\db\Query;

$query = (new Query())->from('post')->where(['status' => 1]); 
```

> Note: If a query already specifies the `orderBy` clause, the new ordering instructions given by end users
  (through the `sort` configuration) will be appended to the existing `orderBy` clause. Any existing `limit`
  and `offset` clauses will be overwritten by the pagination request from end users (through the `pagination` configuration). 

> 注意：如果查询已经定义了 `orderBy` 子句，那么由终端用户（通过 `sort` 设置）给定的新排序参数会附加到现有的 `orderBy` 子句后面。任何现有 `limit` 和 `offset` 子句会被来自终端用户（通过 `pagination` 设置）的分页请求重写。

By default, [[yii\data\ActiveDataProvider]] uses the `db` application component as the database connection. You may
use a different database connection by configuring the [[yii\data\ActiveDataProvider::db]] property.

默认情况下，[[yii\data\ActiveDataProvider]] 使用 `db` 应用程序组件来作为数据库连接器。你可以配置 [[yii\data\ActiveDataProvider]] 属性来使用不同的数据库连接器


## SQL Data Provider <span id="sql-data-provider"></span>

[[yii\data\SqlDataProvider]] works with a raw SQL statement which is used to fetch the needed
data. Based on the specifications of [[yii\data\SqlDataProvider::sort|sort]] and 
[[yii\data\SqlDataProvider::pagination|pagination]], the provider will adjust the `ORDER BY` and `LIMIT`
clauses of the SQL statement accordingly to fetch only the requested page of data in the desired order.

[[yii\data\SqlDataProvider]] 和被用来获取需要的数据的原始 SQL 语句一起工作。在基于 [[yii\data\SqlDataProvider::sort|sort]] 和 [[yii\data\SqlDataProvider::pagination|pagination]] 的说明下，数据提供者会调整 SQL 语句的 `ORDER BY` 和 `LIMIT` 子句，然后根据这个语句仅获取按要求排序数据的被请求页面。

To use [[yii\data\SqlDataProvider]], you should specify the [[yii\data\SqlDataProvider::sql|sql]] property as well
as the [[yii\data\SqlDataProvider::totalCount|totalCount]] property. For example,

使用 [[yii\data\SqlDataProvider]]，你可以定义 [[yii\data\SqlDataProvider::sql|sql]] 属性以及 [[yii\data\SqlDataProvider::totalCount|totalCount]] 属性。比如说，

```php
use yii\data\SqlDataProvider;

$count = Yii::$app->db->createCommand('
    SELECT COUNT(*) FROM post WHERE status=:status
', [':status' => 1])->queryScalar();

$provider = new SqlDataProvider([
    'sql' => 'SELECT * FROM post WHERE status=:status',
    'params' => [':status' => 1],
    'totalCount' => $count,
    'pagination' => [
        'pageSize' => 10,
    ],
    'sort' => [
        'attributes' => [
            'title',
            'view_count',
            'created_at',
        ],
    ],
]);

// returns an array of data rows
$models = $provider->getModels();
```

> Info: The [[yii\data\SqlDataProvider::totalCount|totalCount]] property is required only if you need to
  paginate the data. This is because the SQL statement specified via [[yii\data\SqlDataProvider::sql|sql]]
  will be modified by the provider to return only the currently requested page of data. The provider still
  needs to know the total number of data items in order to correctly calculate the number of pages available.

> 注释：[[yii\data\SqlDataProvider::totalCount|totalCount]] 属性仅当在你需要分页请求数据的时候才会被请求。这是因为通过 [[yii\data\SqlDataProvider::sql|sql]] 定义的 SQL 语句会被可以返回当前请求页面数据的数据提供者修改。数据提供者仍需知道数据项的总数以便正确计算有效页面的数量。


## Array Data Provider <span id="array-data-provider"></span>
## 数组数据提供者 <span id="array-data-provider"></span>

[[yii\data\ArrayDataProvider]] is best used when working with a big array. The provider allows you to return
a page of the array data sorted by one or multiple columns. To use [[yii\data\ArrayDataProvider]], you should
specify the [[yii\data\ArrayDataProvider::allModels|allModels]] property as the big array.
Elements in the big array can be either associative arrays
(e.g. query results of [DAO](db-dao.md)) or objects (e.g. [Active Record](db-active-record.md) instances).
For example,

[[yii\data\ArrayDataProvider]] 是运行大数组（big array）的最佳使用方式。这个数据提供者允许你返回以一个或多个字段排序的数组页。使用 [[yii\data\ArrayDataProvider]]，你应该定义 [[yii\data\ArrayDataProvider::allModels|allModels]] 属性来作为大数组（big array）。
大数组元素要么是关联数组（例子：[DAO](db-dao.md) 查询结果）要么是对象（例子：[Active Record](db-active-record.md) 对象）。
比如，
```php
use yii\data\ArrayDataProvider;

$data = [
    ['id' => 1, 'name' => 'name 1', ...],
    ['id' => 2, 'name' => 'name 2', ...],
    ...
    ['id' => 100, 'name' => 'name 100', ...],
];

$provider = new ArrayDataProvider([
    'allModels' => $data,
    'pagination' => [
        'pageSize' => 10,
    ],
    'sort' => [
        'attributes' => ['id', 'name'],
    ],
]);

// get the rows in the currently requested page
// 得到当前请求页的数据
$rows = $provider->getModels();
``` 

> Note: Compared to [Active Data Provider](#active-data-provider) and [SQL Data Provider](#sql-data-provider),
  array data provider is less efficient because it requires loading *all* data into the memory.

> 注意：用数组数据提供者（array data provider）来比较 [Active Data Provider](#active-data-provider) 和 [SQL Data Provider](#sql-data-provider) 是低效的，因为它需要将*所有*数据加载到内存。

## Working with Data Keys <span id="working-with-keys"></span>
## 使用 Data Keys <span id="working-with-keys"></span>


When using the data items returned by a data provider, you often need to identify each data item with a unique key.
For example, if the data items represent customer information, you may want to use the customer ID as the key
for each customer data. Data providers can return a list of such keys corresponding with the data items returned 
by [[yii\data\DataProviderInterface::getModels()]]. For example,

使用由数据提供者返回的数据项时，你需要频繁地定义那些带着唯一键的数据项。
比如，如果数据项表示用户信息，你也许想使用用户 ID 来作为每个用户数据的 key。数据提供者可以返回一个这样的 key 所对应的，由 [[yii\data\DataProviderInterface::getModels()]] 所返回的数据项的列表。比如，
```php
use yii\data\ActiveDataProvider;

$query = Post::find()->where(['status' => 1]);

$provider = new ActiveDataProvider([
    'query' => $query,
]);

// returns an array of Post objects
// 返回一个 Post 对象数组
$posts = $provider->getModels();

// returns the primary key values corresponding to $posts
// 返回与 $posts 相应的主键值
$ids = $provider->getKeys();
```

In the above example, because you provide to [[yii\data\ActiveDataProvider]] an [[yii\db\ActiveQuery]] object,
it is intelligent enough to return primary key values as the keys. You may also explicitly specify how the key
values should be calculated by configuring [[yii\data\ActiveDataProvider::key]] with a column name or
a callable calculating key values. For example,

在以上例子中，你给 [[yii\data\ActiveDataProvider]] 提供了一个 [[yii\db\ActiveQuery]] 对象，它作为一个 key 来返回主键值是足够智能的。你也可以明确地定义键值应该怎样被计算（通过配置 [[yii\data\ActiveDataProvider::key]]，以列名或可调用对象来计算键值）。比如，

```php
// use "slug" column as key values
// 使用 “slug” 列名作为键值
$provider = new ActiveDataProvider([
    'query' => Post::find(),
    'key' => 'slug',
]);

// use the result of md5(id) as key values
// 使用 md5(id) 的结果作为键值
$provider = new ActiveDataProvider([
    'query' => Post::find(),
    'key' => function ($model) {
        return md5($model->id);
    }
]);
```


## Creating Custom Data Provider <span id="custom-data-provider"></span>
## 建立自定义数据提供者 <span id="custom-data-provider"></span>

To create your own custom data provider classes, you should implement [[yii\data\DataProviderInterface]].
An easier way is to extend from [[yii\data\BaseDataProvider]] which allows you to focus on the core data provider
logic. In particular, you mainly need to implement the following methods:
建立你自己独有的数据提供者类，你应该实现 [[yii\data\DataProviderInterface]] 接口。一个容易的方法是扩展 [[yii\data\BaseDataProvider]]，它允许你聚焦在核心数据提供者逻辑上。实际上，你只需要实现以下方法就可以了：
                                                   
- [[yii\data\BaseDataProvider::prepareModels()|prepareModels()]]: prepares the data models that will be made 
  available in the current page and returns them as an array.
- [[yii\data\BaseDataProvider::prepareModels()|prepareModels()]]：预先准备数据模型，它会在当前页面被有效创建，并且以数组的方式返回它们。
- [[yii\data\BaseDataProvider::prepareKeys()|prepareKeys()]]: accepts an array of currently available data models
  and returns keys associated with them.
- [[yii\data\BaseDataProvider::prepareKeys()|prepareKeys()]]：接收一个当前有效数据模型的数组并且返回和它们相关联的键。
  
- [[yii\data\BaseDataProvider::prepareTotalCount()|prepareTotalCount]]: returns a value indicating the total number 
  of data models in the data provider.
- [[yii\data\BaseDataProvider::prepareTotalCount()|prepareTotalCount]]：返回一个数值，它表示在数据提供者中数据模型的总数。

Below is an example of a data provider that reads CSV data efficiently:
以下是一个数据提供者的例子，它有效地读取　CSV 数据：

```php
<?php
use yii\data\BaseDataProvider;

class CsvDataProvider extends BaseDataProvider
{
    /**
     * @var string name of the CSV file to read
     */
      /**
     * @var string 读取 CSV 文件的名称
     */
    public $filename;
    
    /**
     * @var string|callable name of the key column or a callable returning it
     */
     /**
     * @var string|callable 键列或可以返回值的可调用对象的名称
     */
    public $key;
    
    /**
     * @var SplFileObject
     */
    protected $fileObject; // SplFileObject is very convenient for seeking to particular line in a file
    // SplFileObject 对象在寻找一个文件中的特殊行非常方便
 
    /**
     * @inheritdoc
     */
    public function init()
    {
        parent::init();
        
        // open file
        $this->fileObject = new SplFileObject($this->filename);
    }
 
    /**
     * @inheritdoc
     */
    protected function prepareModels()
    {
        $models = [];
        $pagination = $this->getPagination();
 
        if ($pagination === false) {
            // in case there's no pagination, read all lines
            while (!$this->fileObject->eof()) {
                $models[] = $this->fileObject->fgetcsv();
                $this->fileObject->next();
            }
        } else {
            // in case there's pagination, read only a single page
            $pagination->totalCount = $this->getTotalCount();
            $this->fileObject->seek($pagination->getOffset());
            $limit = $pagination->getLimit();
 
            for ($count = 0; $count < $limit; ++$count) {
                $models[] = $this->fileObject->fgetcsv();
                $this->fileObject->next();
            }
        }
 
        return $models;
    }
 
    /**
     * @inheritdoc
     */
    protected function prepareKeys($models)
    {
        if ($this->key !== null) {
            $keys = [];
 
            foreach ($models as $model) {
                if (is_string($this->key)) {
                    $keys[] = $model[$this->key];
                } else {
                    $keys[] = call_user_func($this->key, $model);
                }
            }
 
            return $keys;
        } else {
            return array_keys($models);
        }
    }
 
    /**
     * @inheritdoc
     */
    protected function prepareTotalCount()
    {
        $count = 0;
 
        while (!$this->fileObject->eof()) {
            $this->fileObject->next();
            ++$count;
        }
 
        return $count;
    }
}
```
