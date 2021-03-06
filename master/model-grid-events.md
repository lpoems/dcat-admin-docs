# 数据表格事件

### 初始化


通过 `Grid::resolving` 方法可以监听表格初始化事件。


开发者可以在这两个事件中改变 `Grid` 的一些设置或行为，比如需要禁用掉某些操作，可以在 `app/Admin/bootstrap.php` 加入下面的代码：

```php
use Dcat\Admin\Grid;

Grid::resolving(function (Grid $grid) {
    $grid->disableActions();

    $grid->disablePagination();

    $grid->disableCreateButton();

    $grid->disableFilter();

    $grid->disableRowSelector();

    $grid->disableTools();

    $grid->disableExport();
});


// 只需要监听一次
Grid::resolving(function (Grid $grid) {
    ...
}, true);
```
这样就不用在每一个控制器的代码中来设置了。

如果全局设置后，要在其中某一个表格中开启设置，比如开启显示操作列，在对应的实例上调用 `$grid->disableActions(false);` 就可以了


### 构建

通过 `Grid::composing` 方法可以监听表格被调用事件。

```php
Grid::composing(function (Grid $grid) {
    ...
});

// 只需要监听一次
Grid::composing(function (Grid $grid) {
    ...
}, true);
```

### 获取数据之前

通过 `Grid::fetching` 方法可以监听表格获取数据之前事件，此事件在 `composing` 事件之后触发。

```php

$grid = new Grid(...);

$grid->fetching(function () {
    ...
});


// 可以在 composing 事件中使用
Grid::composing(function (Grid $grid) {
    $grid->fetching(function (Grid $grid) {
        ...
    });
});
```

### 获取数据之后

通过 `Grid::rows` 方法可以监听表格获取数据之后事件。

```php
use Dcat\Admin\Grid\Row;
use Illuminate\Support\Collection;

$grid->rows(function (Collection $rows) {
    /**
     * 获取第一行数据
     *
     * @var Row $firstRow
     */
    $firstRow = $rows->first();
    
    if ($firstRow) {
        // 获取第一行的 id
        $id = $firstRow->id;
        // 转化为数组
        $row = $firstRow->toArray();
    }
});
```
