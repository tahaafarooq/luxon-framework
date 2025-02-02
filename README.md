*Better documentation coming in the future...*

# What is Luxon?
Luxon is powerful and minimal framework and provides a solid base for your next website. All Luxon's core modules are JSdoc'ed so that your code editor can provide you with helpful information about each method (I use Visual Studio Code and it supports JSdoc).

### :dart: Prerequisites
- PHP 7.4 or newer
- php-mysqli

### :rocket: Installation
- Download this repo as zip file and copy the files in folder `luxon-framework-main` to your web server's document root (which hopefully is empty)
```bash
# or install luxon from terminal (make sure that '/var/www/html' is your web server's document root and that it is empty)
cd /var/www/html
git clone https://github.com/UnrealSecurity/luxon-framework.git .
chmod -R 777 .
```
- Change the default value of **APP_SECRET** in `config/application.php`
- If your website does not support secure connection through HTTPS make sure to set **APP_REQUIRE_HTTPS** to **false** in `config/application.php`
- Configure MySQL database connection and enable it if you need it in `config/database.php`

**Note:** Luxon's loader will try to load PHP files from certain predefined directories and will make them if they don't exist (for example, `controllers` and `models`). If the loader fails to create a missing directory it will throw an error and you'll see `500 - Internal Server Error` in your web browser. (check directory permissions!)
<br/><br/>
If your .htaccess file isn't working you should make sure that your configuration is correct. For apache web server you should check `/etc/apache2/apache2.conf` and ensure that the Directory directive for your document root (typically /var/www/html) has `AllowOverride All` and `Options -Indexes` like in the example below.
```apache
<Directory /var/www/html>
        Options -Indexes
        AllowOverride All
        Require all granted
</Directory>
```

#### Example configuration for NGINX
```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        server_name _;

        try_files /index.php?$query_string /index.php?$query_string;

        location /index.php {
                include snippets/fastcgi-php.conf;
                include fastcgi_params;
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }
}
```

### :truck: Features
- Lightning fast routing
- Database query builder, ORM and templated queries
- Added security

### :fire: Extras
- Enable OPcache in PHP config (php.ini) for extra performance

# 1. Router and routes
Router is one of Luxon's core modules that is used to route incoming request to handler that then takes care of that request. Routes are checked from bottom to top.

### Adding new routes
For example, to route GET requests to our front page we could use something like this
```php
<?php

Router::route("GET", "/^\/$/", function() {
    view('index');
});
```

If we have a controller named `FrontController` with static function named `get_frontPage` we could tell Router that we want that function handle this request
```php
<?php

Router::route("GET", "/^\/$/", ['FrontController', 'get_frontPage']);
```

We can also specify capture groups if we want to extract certain values from the requested URI. This example shows how you can get product category, subcategory and optional page number from the URI.
This would handle requests like `GET /products/office-supplies/chairs/12/` and `GET /products/office-supplies/chairs/`. I do recommend you use separate controllers with more complex routes like this one.
```php
<?php

Router::route("GET", "/^\/products\/([\w\-\_]*)\/([\w\-\_]*)\/?(\d*?)\/?$/", function($maincat, $subcat, $pagenum) {
    // handle request here
    if ($pagenum === "") $pagenum = 0;
    // more code here ...
});
```

# 2. Controllers
In MVC (model-view-controller) model the controller responds to the user input and performs interactions on the data model objects. The controller receives the input, optionally validates it and then passes the input to the model. **Below is a simple login & registration example with Luxon**. You could also create a separate model for these user specific database operations and then from your controller call the user model's methods.

**/routes/routes.php**
```php
<?php

Router::route("GET", "/^\/$/", ['DemoController', 'viewIndex']);
Router::route("POST", "/^\/login\/?$/", ['DemoController', 'doLogin']);
Router::route("POST", "/^\/register\/?$/", ['DemoController', 'doRegister']);
```

**/controllers/DemoController.php**
```php
<?php

    class DemoController {

        public static function viewIndex() {
            if (Session::has('user')) {
                view('clientarea');
            } else {
                view('index');
            }
        }

        public static function doLogin() {
            $username = $_POST['username'];
            $password = $_POST['password'];

            // get one record from table 'users' where username equals $username
            $result = ORM::instance()
                ->select('users')
                ->where(['username', $username])
                ->limit(1)
                ->exec();

            // make sure the QueryResult isn't an error
            if (!$result->isError) {
                // fetch one record from QueryResult and make sure we actually got a record
                if (($row = $result->fetch()) !== null) {
                    // test password
                    if (Password::verify($password, $row['password'])) {
                        Session::set('user', $row);
                    }
                }
            }

            // redirect user back to index
            header('Location: /');
        }

        public static function doRegister() {
            $username = $_POST['username'];
            $email = $_POST['email'];
            $password = $_POST['password'];

            $hash = Password::hash($password);

            $result = ORM::instance()
                ->insert('users', [
                    'username'  => $username,
                    'email'     => $email,
                    'password'  => $hash
                ])->exec();

            // do something with $result?
            // ...

            // redirect user back to index
            header('Location: /');
        }

    }
```

# 3. Models
Coming soon...

# 4. Templated queries
You can use this method `Database::template(string $template, mixed[] ...$data)` to build safe SQL query strings.
```php
// example template
$pageOffset = 0;
$pageSize = 50;

$query = Database::template("SELECT * FROM requests ORDER BY Id DESC LIMIT &", [$pageOffset, $pageSize]);

// Value of $query would then be
// SELECT * FROM requests ORDER BY Id DESC LIMIT 0, 50
// which can be executed safely
$result = Database::query($query);
```

### Symbols and their behavior

| Symbol 	| Data (mixed) 	| Result (string) 	|
|-:	|-	|-	|
| & 	| `'a'` 	| `a` 	|
|  	| `['a', 'b']` 	| `a, b` 	|
| $ 	| `'a'` 	| `a` 	|
|  	| `['a', 'b']` 	| `(a, b)` 	|
| ? 	| `'a'` 	| `'a'` 	|
|  	| `['a', 'b']` 	| `('a', 'b')` 	|
|  	| `[['a', 'b'], ['a', 'b']]` 	| `('a', 'b'), ('a', 'b')` 	|