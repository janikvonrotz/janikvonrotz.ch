---
title: 'Let’s write a Laravel application - Sortable table'
date: 2015-02-04T14:33:28+00:00
author: Janik von Rotz
permalink: /2015/02/04/lets-write-a-laravel-application-sortable-table/
dsq_thread_id:
  - "3484665405"
image: /wp-content/uploads/2015/01/laravel-logo-e1422466263489.png
categories:
  - Laravel
tags:
  - laravel
  - sortable
  - table
  - tutorial
---
I want to show how you can create a sortable table and reuse the code with any model you want to display.
As we don't want to write any function or definition twice or more, let's extend the BaseController class with the function to get sorted items from a variable model.
<!--more-->
**app/controllers/BaseController.php**

```php
class BaseController extends Controller {

  protected  $sortby;
  protected  $order;

  protected function getSortedItems($model)
  {
    $this->sortby = Input::get('sortby');
    $this->order = Input::get('order');
    if ($this->sortby && $this->order) {
      return $model::orderBy($this->sortby, $this->order)->get();
    } else {
       return $model::all();
    }
  }
}
```

You can now call this function in every controller. Here's an example with a Member controller.

**app/controllers/MemberController.php**

```php
class MemberController extends \BaseController {

  public function index()
  {
    $items = $this->getSortedItems('Member');
    $sortby = $this->sortby;
    $order = $this->order;
    
    $action = Route::currentRouteAction();
    $columns = Member::$columns;
    
    return View::make('members.index', compact('items', 'sortby', 'order', 'columns', 'action'));
  }
}
```

The default.index view is supposed to be used by multiple controllers with different models to display.
This requirement leads to variable header definitions for our sortable table.
 That's why we define the columns we want to display in the model as showed below.

**app/models/Member.php**

```php
class Member extends Eloquent{

  public static $columns = [
    "ID"=>"id",
    "Firstname"=>"firstname",
    "Lastname"=>"lastname",
    "E-Mail"=>"email"
  ];
}
```

From now on you can use the next piece of code to create a sortable table in your views.

**views/members/index.blade.php**

```html
<table class="table">
<tr>
  @foreach(array_keys($columns) as $column)
    <th>
    @if ($sortby == $columns[$column] && $order == 'asc')
    {{link_to_action(
      $action,
      $column, ['sortby' => $columns[$column],'order' => 'desc']
    )}}
    @else
    {{link_to_action(
      $action,
      $column, ['sortby' => $columns[$column],'order' => 'asc']
    )}}
    @endif
    </td>
  @endforeach
</tr>
@foreach($items as $member)
    <tr>
      <td>{{$member->id}}</td>
      <td>{{$member->firstname}}</td>
      <td>{{$member->lastname}}</td>
      <td>{{$member->email}}</td>
    </tr>
@endforeach
</table>
```