
https://labs.infyom.com/laravelgenerator/

https://harish81.github.io/infyom-schema-generator/

https://github.com/yajra/laravel-datatables

https://github.com/viralsolani/laravel-adminpanel

https://github.com/silverbux/laravel-angular-admin

https://github.com/viralsolani/laravel-adminpanel

带页码展开组件：https://learnku.com/articles/40405

Windows 服务器如果安装后没效果，请检查网站根目录下是否存在如图所示的静态资源。如果不存在，请在站点根目录下手动创建这样一个文件夹 public\vendor\zhaiduting\column-relation，然后将vendor\zhaiduting\column-relation\dist下的 relate.js 和 relate.css 复制到这个文件夹下就可以了。

```php
$grid = new Grid(new Category());

$grid->column('name', '类别')
     ->relate('topics', ['id', 'title'=>'话题']);

```

```php
$grid = new Grid(new Role());

$grid->column('name', '角色名称')
    ->relate('users', ['id', 'name'=> '用户'], function($user){
        $user->name=
            "<a target='_blank' href='". route('users.show', $user->id). "'>".
            "<img class='img-thumbnail' width='30px' src='".
            $user->avatar ."'> ".
            $user->name.
            "</a>";
    })
;
```
