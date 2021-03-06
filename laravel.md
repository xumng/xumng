
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
app\Admin\Actions\VerSelector.php
```php
<?php

namespace App\Admin\Actions;

use Session;
use Illuminate\Support\Facades\URL;
use App\Models\Post;
use App\Models\Comment;
use Encore\Admin\Actions\Action;
use Illuminate\Http\Request;
use App\Admin\Selectable\Posts;

class VerSelector extends Action
{
    protected $selector = '.verselector';
    protected $product;
    protected $version;
    public $name = 'Select product and version';

    public function handle(Request $request)
    {
        session(['product_id' => $request->get('product_id'),'version_id' => $request->get('version_id')]);
        //Session::forget('version_id');
        $this->render();
        //return $this->response()->success($request->get('product_id'))->topCenter()->refresh();
        //return $this->response()->redirect(URL::route('admin.home'));
        return $this->response()->location(URL::route('admin.home'));
        //return $this->response()->success($request->get('product_id'))->topCenter()->refresh();
    }

    public function form()
    {
        
        if (Session::has('product_id') && Session::has('version_id'))
        {
            $this->select('product_id', 'product')->options(Post::all()->pluck('title', 'id'))
            ->value(session('product_id'))->rules('required')
            ->load('version_id', 'api/comment');

            $this->select('version_id', 'version')->options(function ($id) {
                return Comment::options($id);
            })->value(session('version_id'))->rules('required');
        }
        else
        {
            $this->select('product_id', 'product')->options(Post::all()->pluck('title', 'id'))->rules('required');
            $this->select('version_id', 'version')->options(Comment::all()->pluck('name', 'id'))->rules('required');
        }
    }

    public function html()
    {
        if (Session::has('product_id') && Session::has('version_id'))
        {
            $this->event = 'click';
            $this->product = Post::find(session('product_id'))->title;
            $this->version = Comment::find(session('version_id'))->name;
            return <<<HTML
<li>
    <a class="verselector" href="javascript:void(0);">
      <i class="fa fa-window-restore"></i>
      <span>Product:{$this->product } </span>
      <i class="fa fa-chevron-right" aria-hidden="true"></i>
      <span>Version:{$this->version} Change </span>
      <i class="fa fa-angle-double-down"></i>
    </a>
</li>
HTML;
        }
        else
        {
            $this->product = '';
            $this->version = '';
            return <<<HTML
<li>
    <a class="verselector" href="javascript:void(0);">
      <i class="fa fa-window-restore"></i>
      <span>Product:{$this->product } </span>
      <i class="fa fa-chevron-right" aria-hidden="true"></i>
      <span>Version:{$this->version} Change </span>
      <i class="fa fa-angle-double-down"></i>
    </a>
</li>
HTML;
        }
        
    }
}

```

更简洁的写法
```php
    public function form()
    {
        
        if (Session::has('product_id') && Session::has('version_id'))
        {
            $this->select('product_id', 'product')->options(Post::all()->pluck('title', 'id'))
            ->value(session('product_id'))->rules('required')
            ->load('version_id', 'api/comment');

            // $this->select('version_id', 'version')->options(function ($id) {
            //     return Comment::options($id);
            // })->value(session('version_id'))->rules('required');
            $this->select('version_id', 'version')->options(function ($id) {
                return Comment::Comment()->where('post_id',$id)->pluck('name', 'id');
            })->value(session('version_id'))->rules('required');
        }
        else
        {
            $this->select('product_id', 'product')->options(Post::all()->pluck('title', 'id'))->rules('required');
            $this->select('version_id', 'version')->options(Comment::all()->pluck('name', 'id'))->rules('required');
        }
    }
```

bootstrap.php
```php
    $navbar->right(new Actions\VerSelector());
    $navbar->right(new \App\Admin\Extensions\Nav\Links());
    
```   

app\Admin\Controllers\CommentController.php
```php
    public function comment(Request $request)
    {
        $postId = $request->get('q');

        return Comment::Comment()->where('post_id',$postId)->get(['id', DB::raw('name as text')]);
    }
```

创建comment的方法
app\Models\Comment.php
```php
    public function scopeComment($query)
    {
        return $query;
    }
    public static function options($id)
    {
        if (! $self = static::find($id)) {
            return [];
        }

        return $self->where('post_id',$id)->pluck('name', 'id');
    }
```

解决modal 窗口联动问题
vendor\encore\laravel-admin\resources\views\actions\form\modal.blade.php

```php
<div class="modal" tabindex="-1" role="dialog" id="{{ $modal_id }}">
    <div class="modal-dialog {{ $modal_size }}" role="document">
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
                <h4 class="modal-title">{{ $title }}</h4>
            </div>
            <form class="fields-group">
            <div class="modal-body">
                @foreach($fields as $field)
                    {!! $field->render() !!}
                @endforeach
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default" data-dismiss="modal">{{ __('admin.close') }}</button>
                <button type="submit" class="btn btn-primary">{{ __('admin.submit') }}</button>
            </div>
            </form>
        </div><!-- /.modal-content -->
    </div><!-- /.modal-dialog -->
</div><!-- /.modal -->
```

编辑页面显示列表
```php

    /**
     * Edit interface.
     *
     * @param $id
     * @return Content
     */
    public function edit($id, Content $content)
    {
        return $content
            ->title('header')
            ->description('description')
            ->row($this->form()->edit($id))
            ->row(Admin::grid(PostComment::class, function (Grid $grid) use ($id) {

                $grid->setName('comments')
                    ->setTitle('Comments')
                    ->setRelation(Post::find($id)->comments())
                    ->resource('/demo/post-comments');

                $grid->id();
                $grid->content();
                $grid->created_at();
                $grid->updated_at();

            }));
    }
```   

使用模态窗口编辑
https://learnku.com/docs/dcat-admin/1.x/tools-form/8125

laravel admin 也支持feildset布局，具体可以参考：
https://learnku.com/docs/dcat-admin/1.x/table-layout/8822#35e387
