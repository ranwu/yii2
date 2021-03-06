数据提供者
==============

在[分页](output-pagination.md)和[排序](output-sorting.md)章节里，我们讲解了如何让终端用户去选择某数据的特殊页面以及通过一些字段来让它们排序。由于数据分页和排序是非常普遍的任务，所以 Yii 提供了一套*数据提供者（data provider）*  类来封装它。

数据提供者（data provider）是一个实现 [[yii\data\DataProviderInterface]] 接口的类，主要支持分页检索和数据排序，常常与 [data widgets](output-data-widgets.md) 一起运行以达到终端用户可以交互式地进行分页和排序数据的目的。

以下是包含在 Yii 发行中的数据提供者（data provider）类：

* [[yii\data\ActiveDataProvider]]：使用 [[yii\db\Query]] 或 [[yii\db\ActiveQuery]] 来查询数据库并且以数组或 [Active Record](db-active-record.md) 实例来返回。
* [[yii\data\SqlDataProvider]]：执行 SQL 语句和返回以数组形式的数据库数据。
* [[yii\data\ArrayDataProvider]]：获得一个大数组和返回一部分基于分页和排序的详细说明

所有这些数据提供者的使用方式分为以下常见模式：

```php
// 通过配置它的分页和排序属性来创建数据提供者
$provider = new XyzDataProvider([
    'pagination' => [...],
    'sort' => [...],
]);

// 检索分页和排序数据
$models = $provider->getModels();

// 得到当前页面数据项的数目
$count = $provider->getCount();

// 得到所有页面数据项的总数目
$totalCount = $provider->getTotalCount();
```

你可以配置 [[yii\data\BaseDataProvider::pagination|pagination]] 和 [[yii\data\BaseDataProvider::sort|sort]] 属性来规定数据提供者的分页和排序行为，同时它们也分别对应到 [[yii\data\Pagination]] 和 [[yii\data\Sort]] 的配置。你还可以把它们配置为 `false` 来同时（或单独）关闭分页和排序功能。 

数据小部件 [Data widgets](output-data-widgets.md)，比如 [[yii\grid\GridView]]，有一个称为 `dataProvider` 的属性，它可以获取数据提供者实例以及显示它提供的数据。比如，

```php
echo yii\grid\GridView::widget([
    'dataProvider' => $dataProvider,
]);
```

这些数据提供者主要以源数据的定义方式而变化。在以下子章节中，我们会说明每个数据提供者的详细使用方法。


## 活动数据提供者 <span id="active-data-provider"></span> 

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

// 返回 Post 对象的数组
$posts = $provider->getModels();
```

如果以上例子的 `$query` 变量是通过使用以下代码创建的话，那么这个数据提供者会以原始数组的方式返回。 

```php
use yii\db\Query;

$query = (new Query())->from('post')->where(['status' => 1]); 
```

> 注意：如果查询已经定义了 `orderBy` 子句，那么由终端用户（通过 `sort` 设置）给定的新排序参数会附加到现有的 `orderBy` 子句后面。任何现有 `limit` 和 `offset` 子句会被来自终端用户（通过 `pagination` 设置）的分页请求重写。

默认情况下，[[yii\data\ActiveDataProvider]] 使用 `db` 应用程序组件来作为数据库连接器。你可以配置 [[yii\data\ActiveDataProvider]] 属性来使用不同的数据库连接器


## SQL 数据提供者 <span id="sql-data-provider"></span>

[[yii\data\SqlDataProvider]] 和被用来获取需要数据的原始 SQL 语句一起工作。在基于 [[yii\data\SqlDataProvider::sort|sort]] 和 [[yii\data\SqlDataProvider::pagination|pagination]] 的说明下，数据提供者会调整 SQL 语句的 `ORDER BY` 和 `LIMIT` 子句，然后根据这个语句仅获取按要求排序数据的被请求页面。

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

// 返回一行数据数组
$models = $provider->getModels();
```

> 注释：[[yii\data\SqlDataProvider::totalCount|totalCount]] 属性仅当在你需要分页请求数据的时候才会被请求。这是因为通过 [[yii\data\SqlDataProvider::sql|sql]] 定义的 SQL 语句会被可以返回当前请求页面数据的数据提供者修改。数据提供者仍需知道数据项的总数以便正确计算有效页面的数量。


## 数组数据提供者 <span id="array-data-provider"></span>

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

// 得到当前请求页的数据
$rows = $provider->getModels();
``` 

> 注意：用数组数据提供者（array data provider）来比较 [Active Data Provider](#active-data-provider) 和 [SQL Data Provider](#sql-data-provider) 是低效的，因为它需要将*所有*数据加载到内存。

## 使用 Data Keys <span id="working-with-keys"></span>

使用由数据提供者返回的数据项时，你需要频繁地定义那些带着唯一键的数据项。
比如，如果数据项表示用户信息，你也许想使用用户 ID 来作为每个用户数据的 key。数据提供者可以返回一个这样的 key 所对应的，由 [[yii\data\DataProviderInterface::getModels()]] 所返回的数据项的列表。比如，
```php
use yii\data\ActiveDataProvider;

$query = Post::find()->where(['status' => 1]);

$provider = new ActiveDataProvider([
    'query' => $query,
]);

// 返回一个 Post 对象数组
$posts = $provider->getModels();

// 返回与 $posts 相应的主键值
$ids = $provider->getKeys();
```

在以上例子中，你给 [[yii\data\ActiveDataProvider]] 提供了一个 [[yii\db\ActiveQuery]] 对象，它作为一个 key 来返回主键值是足够智能的。你也可以明确地定义键值应该怎样被计算（通过配置 [[yii\data\ActiveDataProvider::key]]，以列名或可调用对象来计算键值）。比如，

```php
// 使用 “slug” 列名作为键值
$provider = new ActiveDataProvider([
    'query' => Post::find(),
    'key' => 'slug',
]);

// 使用 md5(id) 的结果作为键值（译者注：md5 就是可调用对象）
$provider = new ActiveDataProvider([
    'query' => Post::find(),
    'key' => function ($model) {
        return md5($model->id);
    }
]);
```


## 建立自定义数据提供者 <span id="custom-data-provider"></span>

建立你自己独有的数据提供者类，你应该实现 [[yii\data\DataProviderInterface]] 接口。一个容易的方法是扩展 [[yii\data\BaseDataProvider]]，它允许你聚焦在核心数据提供者逻辑上。实际上，你只需要实现以下方法就可以了：
                                                   
- [[yii\data\BaseDataProvider::prepareModels()|prepareModels()]]：预先准备数据模型，它会在当前页面被有效创建，并且以数组的方式返回它们。
- [[yii\data\BaseDataProvider::prepareKeys()|prepareKeys()]]：接收一个当前有效数据模型的数组并且返回和它们相关联的键。
- [[yii\data\BaseDataProvider::prepareTotalCount()|prepareTotalCount]]：返回一个数值，它表示在数据提供者中数据模型的总数。

以下是一个数据提供者的例子，它有效地读取 CSV 数据：

```php
<?php
use yii\data\BaseDataProvider;

class CsvDataProvider extends BaseDataProvider
{
    /**
     * @var string 读取 CSV 文件的名称
     */
    public $filename;
    
    /**
     * @var string|callable 键列或可以返回值的可调用对象的名称
     */
    public $key;
    
    /**
     * @var SplFileObject
     */
    protected $fileObject; // SplFileObject 对象在寻找一个文件中的特殊行非常方便 
 
    /**
     * @inheritdoc
     */
    public function init()
    {
        parent::init();
        
        // 打开文件
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
            // 这个例子没有分页，读取所有行
            while (!$this->fileObject->eof()) {
                $models[] = $this->fileObject->fgetcsv();
                $this->fileObject->next();
            }
        } else {
            // 这个例子有分页，只读取单行
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
