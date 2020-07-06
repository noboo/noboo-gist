# slimを触って見た。

### ざっくり階層
```
├── cache
│   └── ************.php
├── composer.json
├── composer.lock
├── Controller
│   └── SampleController.php
├── node_modules/
├── package.json
├── public
│   └── index.php
├── resources
│   └── views
│        └── welcome.blade.php
├── routes
│   └── 省略
├── views
│   └── web.php
├── vendor/
├── webpack.mix.js
└── yarn.lock
```
### slimをインストール
フォルダ作って

```unix:setup.sh
$ mkdir slim
$ cd slim/
$ mkdir public resources routes Controller cache
$ mkdir resources/views
```
ファイル作って

```unix:setup.sh
$ touch Controller/SampleController.php public/index.php resources/views/sample.blade.php routes/web.php
```

slimインストール

```unix:setup.sh
composer require slim/slim "^3.0"
```

### ファイルに書き込み

```php:public/index.php
<?php

require __DIR__ . '/../vendor/autoload.php';
require __DIR__ . '/../Controller/SampleController.php';
require __DIR__ . '/../routes/web.php';
```

```php:Controller/SampleController.php
<?php

use Slim\Container;
use Slim\Http\Request;
use Slim\Http\Response;

class SampleController
{
    private $app;

    public function __construct(Container $app)
    {
        $this->app = $app;
    }

    public function index(Request $request, Response $response)
    {
        $name = $request->getAttribute('name');
        $array = ['test' => "Hello, $name"];
        return $this->app->view->render($response, 'sample', $array);
    }
}
```

```php:routes/web.php
<?php

$config = [
    'settings' => [
        'displayErrorDetails' => true, 
        'renderer'            => [
            'blade_template_path' => __DIR__ . '/../resources/views',
            'blade_cache_path'    => __DIR__ . '/../cache', 
        ],
    ],
];

$app = new \Slim\App($config);

$container = $app->getContainer();

$container['view'] = function ($container) {
    return new \Slim\Views\Blade(
        $container['settings']['renderer']['blade_template_path'],
        $container['settings']['renderer']['blade_cache_path']
    );
};

$app->get('/hello/{name}', SampleController::class . ':index');

$app->run();
```

```php:views/sample.blade.php
{{$test}}
```


### テンプレートエンジンの bladeインストール

```unix:setup.sh
composer require rubellum/slim-blade-view
```

PHP built-in serverを起動してローカル環境確認

```unix:setup.sh
php -S localhost:8080 -t public public/index.php
```
確認先
http://localhost:8080/hello/test 

### webpack面倒なのでlaravelmixインストール

package.json 作って、

```unix:setup.sh
touch package.json
```
ファイルに記入。
ほぼ、laravelからのコピペで、yarnを使うのでちょっと書き換え。

```json:package.json
{
    "private": true,
    "scripts": {
        "dev": "yarn run development",
        "development": "cross-env NODE_ENV=development node_modules/webpack/bin/webpack.js --progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js",
        "watch": "yarn run development -- --watch",
        "watch-poll": "yarn run watch -- --watch-poll",
        "hot": "cross-env NODE_ENV=development node_modules/webpack-dev-server/bin/webpack-dev-server.js --inline --hot --config=node_modules/laravel-mix/setup/webpack.config.js",
        "prod": "yarn run production",
        "production": "cross-env NODE_ENV=production node_modules/webpack/bin/webpack.js --no-progress --hide-modules --config=node_modules/laravel-mix/setup/webpack.config.js"
    },
    "devDependencies": {
        "axios": "^0.18",
        "bootstrap": "^4.0.0",
        "cross-env": "^5.1",
        "jquery": "^3.2",
        "laravel-mix": "^4.0.7",
        "lodash": "^4.17.5",
        "popper.js": "^1.12",
        "resolve-url-loader": "^2.3.1",
        "sass": "^1.15.2",
        "sass-loader": "^7.1.0",
        "vue": "^2.5.17"
    }
}

```
### yarnをインストール

yarnを設定してlaravelmixをインストールして、ホームディレクトリにwebpack.mix.jsをコピペ。

```unix:setup.sh
$ yarn
$ laravel-mix --save-dev
$ cp node_modules/laravel-mix/setup/webpack.mix.js ./
```

コンパイル用のjsとsassのフォルダとファイルを作成

```unix:setup.sh
$ mkdir resources/js resources/sass
```

```unix:setup.sh
$ touch resources/js/app.js resources/sass/app.scss
```
webpack.mix.jsに排出先を書き込み

```js:webpack.mix.js
const mix = require("laravel-mix");

mix.js("resources/js/app.js", "public/js").sass(
    "resources/sass/app.scss",
    "public/css"
);
```
おしまい。
既存のサイトをフレームワークにしたりして遊ぶ。


### あとがき

#### PHP built-in serverでcss、js、画像も表示させる
publicフォルダにrouter.phpを作ってローカルでデザイン確認

```php:public/router.php
<?php

// define environment
$_SERVER["FUEL_ENV"] = 'development';

// static files
if (preg_match('/\.(?:png|jpg|jpeg|gif|css|js).?[0-9]{0,10}$/',
               $_SERVER["REQUEST_URI"])
    or strpos($_SERVER["REQUEST_URI"], 'assets/fonts'))
{
    return false;
}

// requests including '.json'
if (strpos($_SERVER['REQUEST_URI'], '.json'))
{
    $_SERVER["PATH_INFO"] = preg_replace('/\?.+$/', '',
                                          $_SERVER['REQUEST_URI']);
}

require_once __DIR__.'/index.php';
```
ローカルでデザイン確認

```unix:setup.sh
php -S localhost:8080 -t public public/router.php
```
#### レンタルサーバーにアップロードしてみる
フォルダやファイル丸見えなので...
publicフォルダに.htaccessを作る

```htaccess:public/.htaccess
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```
publicフォルダをurlから消す為にホームディレクトリに.htaccessを作る。

```htaccess:/.htaccess
RewriteEngine On
RewriteRule ^(.*) /public/$1
```

***
### memo

反省点
フレームワークも、まだまだ勉強すること沢山だな....


#### 参考にさせて頂いたサイト：ありがとうございます。


* [PHP軽量FrameworkのSlim3](https://qiita.com/Syo_pr/items/b55e18a8361b3ff882b5)
* [Slim](http://www.slimframework.com/docs/v3/start/web-servers.html)
* [laravelmix](https://laravel-mix.com/docs/4.0/installation)


***
