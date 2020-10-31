_source: [steps source](https://www.malasngoding.com/membuat-login-dan-register-laravel/) + [google](www.google.com)_
# Install Composer (package manager for PHP)
* [download & install composer](https://getcomposer.org/doc/00-intro.md)
* if you have installed composer: update it using. 

  ```composer self-update```

# Install Laravel
* Go to htdocs folder of XAMPP
* Type command bellow:

  ```
  composer create-project --prefer-dist laravel/laravel project_name
  ```
  _currently the installed version is Laravel Framework 8.12.1_
  
  _you can check it ```php artisan --version```
* Open project directory and run dev server.

  ```
  cd project_name
  php artisan serve
  ```
* Database Preparation: create mysql database in XAMPP. With type: **utf8_unicode_ci**
* See database/migrations/*_create_users_table.php
* Do migrations for user default table and other

  ```php artisan migrate```

* This will trigger error for your db:
  ```bash
  PDOException::("SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes")
  ```
  *  Go to **config/database.php** and change the engine of mysql to:
     ```php
     'charset' => 'utf8',
     'collation' => 'utf8_unicode_ci',
     ```
     [laravel issue workaround link](https://github.com/laravel/framework/issues/24711)
  * Run command bellow to refresh configuration of laravel
    ```
    php artisan config:cache
    ```
  * Rerun your migration

    ```bash
    D:\Learn\Laravel\MegabitForumApi\belajar_laravel>php artisan migrate
    Migration table created successfully.
    Migrating: 2014_10_12_000000_create_users_table
    Migrated:  2014_10_12_000000_create_users_table (331.52ms)
    Migrating: 2014_10_12_100000_create_password_resets_table
    Migrated:  2014_10_12_100000_create_password_resets_table (377.03ms)
    Migrating: 2019_08_19_000000_create_failed_jobs_table
    Migrated:  2019_08_19_000000_create_failed_jobs_table (317.38ms)
    ```

# Authentication
* [steps](https://www.techiediaries.com/laravel-8-rest-api-crud-mysql/)
