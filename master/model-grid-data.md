# 表格数据源

表格对数据的读取操作并非直接依赖于`Model`，而是依赖数据仓库(`Repository`)，关于数据仓库的详细介绍请参考文档[数据仓库](model-repository.md)。

> {tip} 表格的数据是通过 `Dcat\Admin\Contracts\Repository::get` 接口获取的。


<a name="model"></a>
## 数据来自模型

> {tip} 如果你的数据来自`Model`，那么你也可以直接使用`Model`实例，底层会自动把`Model`转化为数据仓库实例。


当数据源支持`Eloquent Model`时，只需创建一个简单的`Repository`类继承`Dcat\Admin\Repositories\EloquentRepository`即可
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\Movie as MovieModel;

class Movie extends EloquentRepository
{
    // 这里定义你的模型类名
    protected $eloquentClass = MovieModel::class;
    
    // 通过这个方法可以指定查询的字段，默认"*"
    public function getGridColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
}
```


### 直接使用模型
如果你还觉得创建 `Repository` 类麻烦，也可以直接把 `Eloquent Model` 的实例传递到 `Grid` 中，底层会自动把 `Eloquent Model` 转化为 `EloquentRepository` 实例 

```php
use App\Models\Movie as MovieModel;

$grid = new Grid(new MovieModel());

...
```


### 修改来源数据

1、使用 `Grid\Model`
```php
use App\Admin\Repositories\Movie;

$grid = new Grid(new Movie());

// 添加默认查询条件
$grid->model()->where('id', '>', 100);

// 设置初始排序条件
$grid->model()->orderBy('id', 'desc');

...

```
其它查询方法可以参考`eloquent`的查询方法。


2、使用 `Model Query`

```php
use App\Models\Movie as MovieModel;

$grid = new Grid(MovieModel::where('id', '>', 100));

...
```


### 关联数据

有以下三种方式让你的表格支持关联数据


1、使用Repository
```php
use App\Admin\Repositories\Movie;

// 相当于 MovieModel::with('categories')
$grid = new Grid(new Movie(['categories']));

$grid->categories;

...
```


2、使用 `Grid\Model`
```php
use App\Admin\Repositories\Movie;

$grid = new Grid(new Movie());

$grid->model()->with('categories');

$grid->categories;

...
```


3、使用 `Model Query`
```php
use App\Models\Movie as MovieModel;

$grid = new Grid(MovieModel::with('categories'));

$grid->categories;

...
```



<a name="api"></a>
## 数据来自外部API

<a name="example"></a>
### 示例

如果数据是来自外部的API，只需要覆写`Repository`中的`get`方法既可, 具体用法可参考下面的示例，采用`豆瓣电影`API获取并展示数据：

> {tip} 通过`$model->getFilter()->input()`方法可以获取过滤器定义的字段值，请尽量不要通过`request()`方法去获取过滤器筛选参数值，因为当页面中有多个表格时可能无法获取到值。关于过滤器定义请参考[查询过滤](model-grid-filters.md)。

```php
<?php
namespace App\Admin\Repositories;

use Dcat\Admin\Grid;
use Dcat\Admin\Repositories\Repository;
use Illuminate\Pagination\LengthAwarePaginator;

class ComingSoon extends Repository
{
    protected $api = 'https://api.douban.com/v2/movie/coming_soon';
    
    /**
     * 定义主键字段名称 
     * 
     * @return string
     */
    public function getPrimaryKeyColumn()
    {
        return '_id';
    }

    /**
     * 查询表格数据
     *
     * @param Grid\Model $model
     * @return LengthAwarePaginator
     */
    public function get(Grid\Model $model)
    {
        // 当前页数
		$currentPage = $model->getCurrentPage();
		// 每页显示行数
		$perPage = $model->getPerPage();

		// 获取排序字段
		[$orderColumn, $orderType] = $model->getSort();

		// 获取"scope"筛选值
		$city = $model->filter()->input(Grid\Filter\Scope::QUERY_NAME, '广州');
		
		// 如果设置了其他过滤器字段，也可以通过“input”方法获取值，如：
		$title = $model->filter()->input('title');
		if ($title !== null) {
			// 执行你的筛选逻辑
			
		}

		$start = ($currentPage - 1) * $perPage;

		$client = new \GuzzleHttp\Client();

		$response = $client->get("{$this->api}?{$this->apiKey}&city=$city&start=$start&count=$perPage");
		$data = json_decode((string)$response->getBody(), true);

		return $model->makePaginator(
			$data['total'] ?? 0,
			$data['subjects'] ?? []
		);
    }

}
```
<a name="grid-model"></a>
### Grid\Model 常用方法

<a name="getCurrentPage"></a>
#### 获取当前页数 (getCurrentPage)
获取当前页码
- 返回值： `int|null` 如果不允许分页返回null
```php
$page = $model->getCurrentPage();
```

<a name="getPerPage"></a>
#### 获取每页显示行数 (getPerPage)
获取每页显示行数
- 返回值： `int|null` 如果不允许分页返回null
```php
$limit = $model->getPerPage();
```

<a name="getSort"></a>
#### 获取排序字段 (getSort)
获取排序字段
- 返回值： `array` `[$orderColumn, 'desc'|'asc']` || `[null, null]`

```php
// $orderColumn 字段名称，如没有进行排序则为 null
// $orderType 正序或倒序： "desc"、"asc"，如没有进行排序则为 null
list($orderColumn, $orderType) = $model->getSort();
```

<a name="getFilter"></a>
#### 获取过滤器对象 (filter)
获取过滤器对象，通过过滤器对象可以获取到搜索表单的值，用法如下
- 返回值 `Dcat\Admin\Grid\Filter`

```php
// 获取"scope"筛选值
$city = $model->filter()->input(Grid\Filter\Scope::QUERY_NAME, '广州');

// 如果设置了其他过滤器字段，也可以通过“input”方法获取值，如：
$title = $model->filter()->input('title');
if ($title !== null) {
    // 执行你的筛选逻辑
}
```


<a name="sql"></a>
## 数据来自复杂SQL查询

如果来源数据需要执行比较复杂的SQL语句获取，那么有两个办法, 第一个办法就是上面的方法，覆盖掉`Repository`的`get`方法实现


第二个方式是在数据库中建立视图和`model`绑定（未测试过，理论上可行）
