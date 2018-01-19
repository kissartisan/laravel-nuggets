# Handy cheat sheet

### 1. Defining a password mutator

Define a mutator in your model (most likely app\User.php) which would encrypt all your password fields:

    public function setPasswordAttribute($password)
    {   
        $this->attributes['password'] = bcrypt($password);
    }

Now you don't need to write the bcrypt function when dealing with the password field in subsequent controllers.

Found at [Scotch.io tutorial on user authorization](https://scotch.io/tutorials/user-authorization-in-laravel-54-with-spatie-laravel-permission)

### 2. Booting a trait

Replicate the boot method of the class being inject into

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
    
Convention = boot + [name of the trait]
In our example = boot + RecordsActivity

* Note: you should declare it as static method

Found at [Laracast.com - Let's Build A Forum with Laravel and TDD: ](https://laracasts.com/series/lets-build-a-forum-with-laravel/episodes/25)
