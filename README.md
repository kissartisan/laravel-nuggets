# Handy cheat sheet

### 1. Defining a password mutator

Define a mutator in your model (most likely app\User.php) which would encrypt all your password fields:

    public function setPasswordAttribute($password)
    {   
        $this->attributes['password'] = bcrypt($password);
    }

Now you don't need to write the bcrypt function when dealing with the password field in subsequent controllers.

- Found at [Scotch.io tutorial on user authorization](https://scotch.io/tutorials/user-authorization-in-laravel-54-with-spatie-laravel-permission)
