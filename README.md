# Handy cheat sheet

### 1. Defining a password mutator

Define a mutator in your model (most likely app\User.php) which would encrypt all your password fields:

    public function setPasswordAttribute($password)
    {   
        $this->attributes['password'] = bcrypt($password);
    }

Now you don't need to write the bcrypt function when dealing with the password field in subsequent controllers.

Found at [Scotch.io tutorial on user authorization](https://scotch.io/tutorials/user-authorization-in-laravel-54-with-spatie-laravel-permission)


### 2 . Appending accessors

Sometimes, particularly when working with APIs or on other specific instances, you may need to append particular accessors (or attributes) to the default collection returned by Eloquent when retrieving records from the database. 

Ex.

Create a full name accessor:

    class User extends Model
    {
        public function getFullNameAttribute()
        {
            return $this->first_name . ' ' . $this->last_name;
        }
    }
    
Convention = get + [name of your attribute] + Attribute

To access an accessor:

    {{ $user->fullName }}
    
To consume it in an API for example, you need the $appends property on your model to include in the default collection return by Eloquent:

    class User extends Model
    {
        protected $appends = ['fullName'];
    }

Found at [Terry Harvey - 5 Laravel Eloquent Tips & Tricks](https://terryharvey.co.uk/5-laravel-eloquent-tips-tricks/)


### 2. Booting a trait

Replicate the boot method of the class being inject into. 

Ex:

    protected static function bootRecordsActivity()
    {
        if (auth()->guest()) return;

        foreach (static::getActivitiesToRecord() as $event) {
            static::$event(function ($model) use ($event) {
                $model->recordActivity($event);
            });
        }

        static::deleting(function($model) {
            $model->activity()->delete();
        });
    }
    
Convention = boot + [name of your trait]

* Note: you should declare it as static method

Found at [Laracast.com - Let's Build A Forum with Laravel and TDD](https://laracasts.com/series/lets-build-a-forum-with-laravel/episodes/25)


### 3. Creating dropdowns using Eloquent lists method

Suppose you want to show a categories dropdown whose information is pulled from an Eloquent model

    $categories = Category::lists('title', 'id');
    
Then somewhere in your view:

    {{ Form::select('category', $categories) }}
 
Found at [laravel-tricks.com - Easy dropdowns with Eloquent's Lists method](http://laravel-tricks.com/tricks/easy-dropdowns-with-eloquents-lists-method)


### 4. Creating/modifying request input value before custom request validation

If you want to create/modify request input before it gets validated, you can use getValidatorInstance().

    public function getValidatorInstance()
    {
        // Do something here

        return parent::getValidatorInstance();
    }

Then, inside the method, you can execute request merging like so:

    public function getValidatorInstance()
    {
        $this->getPermalink();

        return parent::getValidatorInstance();
    }

    
    /**
     * Get the permalink using title
     */
    public function getPermalink()
    {
        $this->merge(['permalink' => str_slug(request()->input('title'))]);
    }
    
Found at [StackOverflow](https://laracasts.com/discuss/channels/requests/modify-request-input-value-before-validation) while coding my own needs.
