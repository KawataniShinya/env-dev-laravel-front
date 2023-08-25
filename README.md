# env-dev-laravel-front

## overview
ローカルでLaravel開発をするためのdocker環境。<br>
フロント開発もできるよう、Node.jsでホットリロード(npm run dev)できるコンテナを含む。

## composition
- Docker
- Nginx
- MySQL
- Node.js
- Redis

## Usega (example of How to prepare environment for develop)
サーバー側をLaravel、フロントをInertiaで構築する場合の開発環境構築例。

### 1. コンテナ起動
(ローカル)
```shell
docker compose build
docker compose up -d
```

### 2. Laravel インストール
(ローカル)
```shell
docker compose exec app bash
```
(app コンテナ)
```shell
cd /var/www/app
rm .gitignore
composer create-project laravel/laravel:^9 --prefer-dist .
```

### 3. Laravel Breeze インストール
(app コンテナ)
```shell
composer require laravel/breeze:^1
```

### 4. Inertia インストール
(app コンテナ)
```shell
php artisan breeze:install vue
```

### 5. Laravel補完ライブラリ追加
(app コンテナ)

A, B は開発途中で追加がある度に実行。
```shell
composer require --dev barryvdh/laravel-ide-helper
composer require --dev doctrine/dbal
## Facade --- A
php artisan ide-helper:generate
## Model --- B
php artisan ide-helper:model
```

### 6. アプリケーション初期設定
(app コンテナ)
```shell
composer install
php artisan key:generate
npm install
npm run build
exit
```

### 7. データベース作成
(ローカル)
```shell
docker compose exec db bash
```
(db コンテナ)
```shell
mysql -u root -proot -e "CREATE USER 'laravelUser' IDENTIFIED BY 'password000'"
mysql -u root -proot -e "GRANT all ON *.* TO 'laravelUser'"
mysql -u root -proot -e "FLUSH PRIVILEGES"
mysql -u root -proot -e "CREATE DATABASE laravel_sample"
exit
```

### 8. アプリ側データベース接続設定
(ローカル)

app/.envの下記部分を変更
```
# 修正前
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=

# 修正後
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel_sample
DB_USERNAME=laravelUser
DB_PASSWORD=password000
```

### 9. nodeコンテナ用設定
(ローカル)

app/package.json の該当箇所に下記記述を追加。<br>
Node.js サーバー起動時、htmlに展開されるjsへのリンク(宛先ホスト)が、nodeコンテナ配下を示すようになる。
```shell
{
    "scripts": {
        "docker": "vite --host localhost.node.sample.jp --port 80",
    },
}
```

### 10. 初期データ設定
(ローカル)

app/database/seeders/UserSeeder.php を下記内容で追加。
```
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => 'test',
            'email' => 'test@test.com',
            'password' => Hash::make('password123'),
        ]);
    }
}
```
app/database/seeders/DatabaseSeeder.php の該当箇所に下記内容を追加。
```
class DatabaseSeeder extends Seeder
{
    public function run()
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```
```shell
docker compose exec app bash
```
(app コンテナ)
```shell
php artisan migrate:fresh --seed
exit
```

### 11. コンテナ停止
(ローカル)
```shell
docker compose down
```

### 12. hosts設定
(ローカル)

hosts に下記エントリーを追加
```shell
127.0.0.1 localhost.app.sample.jp
127.0.0.1 localhost.node.sample.jp
```

### 13. Node.js 起動モード変更
(ローカル)

docker-compose.yaml の services/node/command の値を下記に変更する。
```
# build:build only(for product), docker:npm run dev on docker, bash:only start container
docker
```

### 14. コンテナ起動
(ローカル)
```shell
docker compose build
docker compose up -d
```

### 15. ログイン
```
http://localhost.app.sample.jp/login
```
```
Email : test@test.com
Password : password123
```

### 16. 開発にあたって
- 以降、Laravelもフロント側も、変更は動的にweb画面に反映される。
- フロントについて`npm run dev`ではなく本番配置用ファイル生成だけをしたい場合は、上記項番10で`build`を指定する。
- セッション管理等でRedis使用の場合は別途設定の必要あり。
  - `composer require predis/predis:2.1`
  - [Redis設定](https://github.com/KawataniShinya/laravel-redis/compare/163ff23ebf09594c763bdc9d269d48ab4144c990..a90689c0ce613b5acdcaf03f85b388cae0fcddd9)