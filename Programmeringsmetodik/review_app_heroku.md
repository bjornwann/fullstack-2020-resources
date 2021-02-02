# Checklista för review apps

## Versionshantera (Git)

1. börja versionshantera laravel-projektet om ni inte redan gjort det (`git init`)
1. skapa en första commit i master/main-branchen (`git add .`)
   (`git commit -m "commit message"`)
1. skapa nytt repo på github
1. lägg till en remote på ert projekt enligt inställningarna på det nya repot på github
1. pusha upp den första commiten

## Heroku setup

1. lägg till en pipline i Heroku och synka den till repot på GitHub
1. skapa en ny branch i ert Laravelprojekt (t ex `git checkout -b review-app`)

## Review app, setup i Laravelprojektet i den nya branchen

### app.json

Skapa en fil som heter app.json i root för projektet

    {
        "addons": [
            "heroku-postgresql"
        ],
        "buildpacks": [
            {
                "url": "heroku/php"
            }
        ],
        "env": {
            "APP_NAME": "my-laravel-app",
            "APP_ENV": "review",
            "APP_DEBUG": "true",
            "DB_CONNECTION": "pgsql",
            "APP_KEY": {
                "generator": "secret"
            }
        }
    }

### composer.json

lägg till "compile" under scripts i composer.json

        "scripts": {
            "compile": [
                "@php artisan route:cache",
                "@php artisan event:cache",
                "@php artisan view:cache"
            ]
        }
    }

### Procfile

Ändra i Procfile (inställningsfil för Heroku) i root i projektet och lägg till följande rad på slutet:
release: 
`php artisan migrate --force && php artisan cache:clear && php artisan config:cache`

### app.php

Ändra i app.php (\config\app.php)

under App URL ersätt `'url' => env('APP_URL', 'http://localhost'),` med följande

    'url' => env('APP_URL', env('HEROKU_APP_NAME') ? 'https://' . env('HEROKU_APP_NAME') . '.herokuapp.com' : 'http://localhost'),

under Encryption key ersätt `'key' => env('APP_KEY'),` med nedan för att unvdika fel med kryptering mellan Heroku och Laravel

    'key' => strpos(env('APP_KEY'), 'base64:') !== false ? env('APP_KEY') : substr(env('APP_KEY'), 0, 32),

### database.php

ändra så att pgsql-delen av database.php (\config\database.php) ser ut enligt nedan

    'pgsql' => [
        'driver' => 'pgsql',
        'url' => env('DATABASE_URL'),
        'host' => env('DB_HOST'),
        'port' => env('DB_PORT'),
        'database' => env('DB_DATABASE'),
        'username' => env('DB_USERNAME'),
        'password' => env('DB_PASSWORD'),
        'charset' => 'utf8',
        'prefix' => '',
        'prefix_indexes' => true,
        'schema' => 'public',
        'sslmode' => 'prefer',
    ],

## Trust proxies

Ändra i TrustProxies.php (\App\Http\Middleware\TrustProxies.php) för att ge stöd för det sättet Heroku behandlar requests

    /**
     * The trusted proxies for this application.
     *
     * @var array|string
     */
    protected $proxies = '*';

    /**
     * The headers that should be used to detect proxies.
     *
     * @var int
     */
    protected $headers = Request::HEADER_X_FORWARDED_AWS_ELB;

## https

För att tvinga Laravel att köra https (löser eventuella problem med CSS och andra resurser som inte läses in över https)
Ändra i AppServiceProvider.php (\app\Providers) i metoden boot (ändra eller lägg till villkor för andra miljöer om det behövs 'byt ut production')

    public function boot()
        {
            if($this->app->environment('production')) {
            \URL::forceScheme('https');
            }
        }

## slutsteg

1. stage och commita ändringarna och pusha den nya branchen till GitHub (`git add .`) (`git commit -m "commit message"`) (`git push [namn på remote, oftast origin] [namn på branch, i mitt fall review-app]`)
1. aktivera review-app i Heroku
1. skapa en pullrequest på GitHub
1. en automatisk review-app ska nu skapas i Heroku för den pullrequesten
1. kolla att den funkar i Heroku när den har byggt klart

## Läs mer här om varför vi gör allt detta och hur det funkar här:

### GitHub integration

https://devcenter.heroku.com/articles/github-integration

### Pipelines

https://devcenter.heroku.com/articles/pipelines

### review-apps

https://devcenter.heroku.com/articles/github-integration-review-apps

### Continuous Delivery

https://devcenter.heroku.com/categories/continuous-delivery
