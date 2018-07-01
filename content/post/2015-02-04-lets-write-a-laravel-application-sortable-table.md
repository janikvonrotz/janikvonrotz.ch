---
id: 2951
title: 'Let&#8217;s write a Laravel application &#8211; Sortable table'
date: 2015-02-04T14:33:28+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2951
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

[code lang="php"]
class BaseController extends Controller {

  protected  $sortby;
  protected  $order;

  protected function getSortedItems($model)
  {
    $this-&gt;sortby = Input::get('sortby');
    $this-&gt;order = Input::get('order');
    if ($this-&gt;sortby &amp;&amp; $this-&gt;order) {
      return $model::orderBy($this-&gt;sortby, $this-&gt;order)-&gt;get();
    } else {
       return $model::all();
    }
  }
}
[/code]

You can now call this function in every controller. Here's an example with a Member controller.

**app/controllers/MemberController.php**

[code lang="php"]
class MemberController extends \BaseController {

  public function index()
  {
    $items = $this-&gt;getSortedItems('Member');
    $sortby = $this-&gt;sortby;
    $order = $this-&gt;order;
    
    $action = Route::currentRouteAction();
    $columns = Member::$columns;
    
    return View::make('members.index', compact('items', 'sortby', 'order', 'columns', 'action'));
  }
}
[/code]

The default.index view is supposed to be used by multiple controllers with different models to display.
This requirement leads to variable header definitions for our sortable table.
 That's why we define the columns we want to display in the model as showed below.

**app/models/Member.php**

[code lang="php"]
class Member extends Eloquent{

  public static $columns = [
    &quot;ID&quot;=&gt;&quot;id&quot;,
    &quot;Firstname&quot;=&gt;&quot;firstname&quot;,
    &quot;Lastname&quot;=&gt;&quot;lastname&quot;,
    &quot;E-Mail&quot;=&gt;&quot;email&quot;
  ];
}
[/code]

From now on you can use the next piece of code to create a sortable table in your views.

**views/members/index.blade.php**

[code lang="html"]
&lt;table class=&quot;table&quot;&gt;
&lt;tr&gt;
  @foreach(array_keys($columns) as $column)
    &lt;th&gt;
    @if ($sortby == $columns[$column] &amp;&amp; $order == 'asc')
    {{link_to_action(
      $action,
      $column, ['sortby' =&gt; $columns[$column],'order' =&gt; 'desc']
    )}}
    @else
    {{link_to_action(
      $action,
      $column, ['sortby' =&gt; $columns[$column],'order' =&gt; 'asc']
    )}}
    @endif
    &lt;/td&gt;
  @endforeach
&lt;/tr&gt;
@foreach($items as $member)
    &lt;tr&gt;
      &lt;td&gt;{{$member-&gt;id}}&lt;/td&gt;
      &lt;td&gt;{{$member-&gt;firstname}}&lt;/td&gt;
      &lt;td&gt;{{$member-&gt;lastname}}&lt;/td&gt;
      &lt;td&gt;{{$member-&gt;email}}&lt;/td&gt;
    &lt;/tr&gt;
@endforeach
&lt;/table&gt;
[/code]