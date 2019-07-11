---
title: 'Let’s write a Laravel application - Fulltext search'
date: 2015-02-13T09:59:27+00:00
author: Janik Vonrotz
slug: lets-write-a-laravel-application-fulltext-search
images:
  - /wp-content/uploads/2015/01/laravel-logo-e1422466263489.png
categories:
  - Web development
tags:
  - function
  - laravel
  - search
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

```php
Route::get('search', [
    'as' => 'search', 'uses' => 'MemberController@search'
]);
```

**app/models/*.php**

For every model that will be searchable you have to add the searchable trait and define the searchable columns and their result relevance. You can get the details of this step from the searchable trait documentation (link above).

You can use any controller of your app to handle the search requests as long you'll add the following function to this controller.

**app/controllers/MemberController.php**

```php
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

		$results = $model::search($query)->get();
		return View::make('search.index', compact('results', 'model', 'route'));
	}
```

For every route you want to provide the search functionality you have to add a case and define the model and redirect route.

Now add a new view to display our search results and customize the look of the search results for every model type.

**app/views/search/index.blade.php**

```php
@extends('default.master')

@section('content')
  @foreach($results as $result)
    <a href="{{ URL::to($route.'/' . $result->id . '/edit') }}">

    @if($model=='Member')
    <article>
      <h2>{{$result->firstname.' '.$result->lastname}}</h2>
      <p>{{$result->id.' '.$result->email}}</p>
    </article>
    @endif

    @if($model=='Category')
    <article>
      <h2>{{$result->name}}</h2>
      <p>{{$result->id}}</p>
    </article>
    @endif

    </a>
  @endforeach
@stop
```

Finally we add the search bar to a default blade template.

**app/views/default/navigation.blade.php**

```php
  {{Form::open(['url' => 'search', 'method' => 'GET','class' => 'navbar-form navbar-right', 'role' => 'search'])}}
      <div class="form-group">
        {{ Form::hidden('route', isset($route) ? $route : Route::getCurrentRoute()->getPath())}}
        {{ Form::text('q', null, ['class' => 'form-control', 'placeholder' => 'Search'])}}
      </div>
      {{ Form::button('', ['type' => 'submit', 'class' => 'btn btn-default glyphicon glyphicon-search']) }}
    {{Form::close()}}
```

If done you're are ready to run.