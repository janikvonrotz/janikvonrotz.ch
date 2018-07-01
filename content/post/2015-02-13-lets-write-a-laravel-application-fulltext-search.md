---
id: 2966
title: 'Let&#8217;s write a Laravel application &#8211; Fulltext search'
date: 2015-02-13T09:59:27+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2966
permalink: /2015/02/13/lets-write-a-laravel-application-fulltext-search/
dsq_thread_id:
  - "3511905348"
image: /wp-content/uploads/2015/01/laravel-logo-e1422466263489.png
categories:
  - Laravel
tags:
  - all
  - columns
  - fulltext
  - function
  - laravel
  - relevance
  - search
  - table
---
This time I want to show you how you add a search functionality to your Laravel application that accomplishes the following features:

* Fulltext search in Eloquent model
* Define searchable columns
* Define result relevance for searchable columns
* Customize the look of the search results based on the model
* Independent search bar.

<!--more-->
To search in Laravel models we will use this [searchable trait](https://github.com/nicolaslopezj/searchable) from [Nicolás López](https://github.com/nicolaslopezj). Install it with composer.

First add a new route to handle our search requests.

**app/routes.php**

[code lang="php"]
Route::get('search', [
    'as' =&gt; 'search', 'uses' =&gt; 'MemberController@search'
]);
[/code]

**app/models/*.php**

For every model that will be searchable you have to add the searchable trait and define the searchable columns and their result relevance. You can get the details of this step from the searchable trait documentation (link above).

You can use any controller of your app to handle the search requests as long you'll add the following function to this controller.

**app/controllers/MemberController.php**

[code lang="php"]
	public function search()
	{
		$query = Input::get('q');
		$route = Input::get('route');

		if(preg_match( '/^members.*/', $route)){
			$model = 'Member';
			$route = 'members';
		}elseif(preg_match( '/^categories.*/', $route)){
			$model = 'Category';
			$route = 'categories';
		}elseif(preg_match( '/^boats.*/', $route)){
			$model = 'Boat';
			$route = 'boats';
		}elseif(preg_match( '/^damages.*/', $route)){
			$model = 'Damage';
			$route = 'damages';
		}elseif(preg_match( '/^trips.*/', $route)){
			$model = 'Trip';
			$route = 'trips';
		}elseif(preg_match( '/^users.*/', $route)){
			$model = 'User';
			$route = 'users';
		}else{
			$model = 'Member';
			$route = 'members';
		}

		$results = $model::search($query)-&gt;get();
		return View::make('search.index', compact('results', 'model', 'route'));
	}
[/code]

For every route you want to provide the search functionality you have to add a case and define the model and redirect route.

Now add a new view to display our search results and customize the look of the search results for every model type.

**app/views/search/index.blade.php**

[code lang="php"]
@extends('default.master')

@section('content')
  @foreach($results as $result)
    &lt;a href=&quot;{{ URL::to($route.'/' . $result-&gt;id . '/edit') }}&quot;&gt;

    @if($model=='Member')
    &lt;article&gt;
      &lt;h2&gt;{{$result-&gt;firstname.' '.$result-&gt;lastname}}&lt;/h2&gt;
      &lt;p&gt;{{$result-&gt;id.' '.$result-&gt;email}}&lt;/p&gt;
    &lt;/article&gt;
    @endif

    @if($model=='Category')
    &lt;article&gt;
      &lt;h2&gt;{{$result-&gt;name}}&lt;/h2&gt;
      &lt;p&gt;{{$result-&gt;id}}&lt;/p&gt;
    &lt;/article&gt;
    @endif

    &lt;/a&gt;
  @endforeach
@stop
[/code]

Finally we add the search bar to a default blade template.

**app/views/default/navigation.blade.php**

[code lang="php"]
  {{Form::open(['url' =&gt; 'search', 'method' =&gt; 'GET','class' =&gt; 'navbar-form navbar-right', 'role' =&gt; 'search'])}}
      &lt;div class=&quot;form-group&quot;&gt;
        {{ Form::hidden('route', isset($route) ? $route : Route::getCurrentRoute()-&gt;getPath())}}
        {{ Form::text('q', null, ['class' =&gt; 'form-control', 'placeholder' =&gt; 'Search'])}}
      &lt;/div&gt;
      {{ Form::button('', ['type' =&gt; 'submit', 'class' =&gt; 'btn btn-default glyphicon glyphicon-search']) }}
    {{Form::close()}}
[/code]

If done you're are ready to run.