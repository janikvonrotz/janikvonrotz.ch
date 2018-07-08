---
title: 'Let’s write a Laravel application - Form validation for your backend and frontend'
date: 2015-02-10T00:18:29+00:00
author: Janik von Rotz
slug: lets-write-a-laravel-application-form-validation-for-your-backend-and-frontend
dsq_thread_id:
  - "3500959987"
image: /wp-content/uploads/2015/01/laravel-logo-e1422466263489.png
categories:
  - Laravel
tags:
  - backend
  - done
  - form
  - frontend
  - jquery
  - laravel
  - right
  - validation
  - validator
---
Form validation is always part of your data process workflow. With Laravel you can to validate the input on the controller or the eloquent model. To provide a seamless user experience it's better to validate a form with JavaScript in the frontend. However due to security reasons you have provide a backend validation, otherwise it would possible to inject non valid data.
<!--more-->
Luckily [Bilal Gültekin](https://github.com/bllim) came up with this awesome [solution](https://github.com/bllim/laravel-to-jquery-validation).

The ideas is simply avoid writing validation rules twice (backend and frontend) and rather reuse the backend validation rules by adding them as parameter to our frontend form. 
The extension requires the [jquery validation plugin](https://github.com/jzaefferer/jquery-validation).

Let's get started by installing the required assets.

Install the Laravel extension with composer.

**composer.json**

```
"require": {
	"bllim/laravel-to-jquery-validation": "*"
}
```

Register the new service provider.

**app/config/app.php**

```
'providers' => array(
	 'Bllim\LaravelToJqueryValidation\LaravelToJqueryValidationServiceProvider',
)
```

The extension comes with a configuration file and two assets. You have to publish them first.

```
php artisan config:publish bllim/laravel-to-jquery-validation
php artisan asset:publish bllim/laravel-to-jquery-validation
```

In the folder **public/packages/bllim/laravel-to-jquery-validation/** you'll find now two JavaScript files. Include them in your form view. You can also use bower to maintain the jquery validation plugin.
In addition you have to add this piece of JavaScript to enable the frontend validation for your form.

```js
$('form').validate();
```

Now that we are ready let's add the validation rules to our model.

**app/model/Member.php**

```php
class Member extends Eloquent{

  public static $rules = [
      'firstname' => 'required',
      'lastname' => 'required',
      'email' => 'required|email'
  ];
}
```

As I've said we have to validate the input data in backend anyway, so here's an example of the store function.

**app/controllers/MemberController.php**

```php
class MemberController extends \BaseController {
	public function store()
	{
		$validator = Validator::make($data = Input::all(), Member::$rules);
	
		if ($validator->fails()) {
			return Redirect::back()->withErrors($validator)->withInput(Input::except('password'));
		}
	
		$member->create($data);
	
		Session::flash('message', 'Successfully created Member!');
		return Redirect::to('members');
	}
]
```

In your form view you can now set the validation rules from your model.

```
{{Form::setValidation(Member::$rules);}}
```

Here's an example of an compatible form definition for this kind of validation.

```
{{ Form::open(array('url' => 'members')) }}
  <div class="form-group">
  {{ Form::label('firstname', 'Firstname') }}
  {{ Form::text('firstname', Input::old('firstname'), array('class' => 'form-control')) }}
  </div>
  <div class="form-group">
  {{ Form::label('lastname', 'Lastname') }}
  {{ Form::text('lastname', Input::old('lastname'), array('class' => 'form-control')) }}
  </div>
  <div class="form-group">
  {{ Form::label('email', 'E-Mail') }}
  {{ Form::text('email', Input::old('email'), array('class' => 'form-control')) }}
  </div>
  {{ Form::submit('Save', array('class' => 'btn btn-primary')) }}
{{ Form::close() }}
```


From now on the input of your form will be validated in the front- and backend with the same rules.