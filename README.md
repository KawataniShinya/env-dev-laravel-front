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
composer create-project laravel/laravel:12.1.0 --prefer-dist .
```

### 3. Laravel Breeze インストール
(app コンテナ)
```shell
composer require laravel/breeze:2.3.8
```

### 4. Inertia インストール
(app コンテナ)
```shell
php artisan breeze:install vue
```

### 5. アプリケーション初期設定
(app コンテナ)
```shell
composer install
php artisan key:generate
exit
```

### 6. フロントエンドビルド
(ローカル)
```shell
docker compose exec node bash
```
(node コンテナ)
```shell
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

### 9. Laravel開発支援ライブラリ追加
(ローカル)
```shell
docker compose exec app bash
```
(app コンテナ)

#### IDE(統合開発環境)の補完や型推論を強化
A, B は開発途中で追加がある度に実行。
```shell
composer require --dev barryvdh/laravel-ide-helper
## Facade --- A
php artisan ide-helper:generate
## Model --- B
php artisan ide-helper:model
exit
```

#### マイグレーション実行時により高度なスキーマ操作を行う
```shell
composer require --dev doctrine/dbal
```

### 10. nodeコンテナ用設定
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

### 11. 初期データ設定
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

### 12. コンテナ停止
(ローカル)
```shell
docker compose down
```

### 13. hosts設定
(ローカル)

hosts に下記エントリーを追加
```shell
127.0.0.1 localhost.app.sample.jp
127.0.0.1 localhost.node.sample.jp
```

### 14. Node.js 起動モード変更
(ローカル)

docker-compose.yaml の services/node/command の値を下記に変更する。
```
# build:build only(for product), docker:npm run dev on docker, bash:only start container
docker
```

### 15. オリジン許可とホットリロード機能有効化
開発環境ではフロントエンドのホットリロードのため、バックエンドとは別コンテナで動作させる。\
その際に発生するCORSエラーを避けるため、`vite.config.js`にヘッダー`access-control-allow-origin: *`を付加する設定を追加。

また、dockerホスト環境ではホットリロードが効かないことがある。\
予防のため`vite.config.js`に`usePolling: true`を追加。
```
export default defineConfig({
    plugins: [
      ...
    ],
    server: {
        cors: {
            origin: '*',
        },
        watch: {
            usePolling: true,
        }
    }
});
```

### 16. セッションRedis化
セッション管理にRedisを利用するように変更。

(ローカル)

app/.envの下記部分を変更
```
#修正前
SESSION_DRIVER=database
REDIS_HOST=127.0.0.1

#修正後
SESSION_DRIVER=redis
REDIS_HOST=redis
```

### 17. コンテナ起動
(ローカル)
```shell
docker compose build
docker compose up -d
```

### 18. ログイン
```
http://localhost.app.sample.jp/login
```
```
Email : test@test.com
Password : password123
```

### 19. セッション保存確認
Redisコンテナにセッションが保存されていることを確認。

(ローカル)
```shell
docker compose exec redis sh
```

(Redis コンテナ)
```shell
redis-cli
> KEYS *
1) "laravel-database-laravel-cache-8ROIIdtNkleH06LckPcIFoHBZwLo9yL3jqJKDozG"
> get laravel-database-laravel-cache-8ROIIdtNkleH06LckPcIFoHBZwLo9yL3jqJKDozG
"s:218:\"a:3:{s:6:\"_token\";s:40:\"LOmQAsgnZ3h9m8d0rdpNkyhDwUozA6DjiqfCrPh5\";s:9:\"_previous\";a:2:{s:3:\"url\";s:36:\"http://localhost.app.sample.jp/login\";s:5:\"route\";s:5:\"login\";}s:6:\"_flash\";a:2:{s:3:\"old\";a:0:{}s:3:\"new\";a:0:{}}}\";"
```

### 20. 開発にあたって
- 以降、Laravelもフロント側も、変更は動的にweb画面に反映される。
- フロントについて`npm run dev`ではなく本番配置用ファイル生成だけをしたい場合は、上記項番10で`build`を指定する。
- リクエストのタイムアウト値はデフォルト30分としているが、変更する場合は下記値を更新。
  - docker-compose.yaml
    - nginx-proxy -> environment -> VIRTUAL_TIMEOUT
  - docker/app/nginx/default.conf
    - fastcgi_connect_timeout 
    - fastcgi_send_timeout
    - fastcgi_read_timeout
    - proxy_connect_timeout
    - proxy_send_timeout
    - proxy_read_timeout
    - send_timeout
    - keepalive_timeout
- Xdebugを利用する場合はクライアント側に設定が必要
  - [Xdebug設定](https://github.com/KawataniShinya/php-sftp-docker/tree/main#phpstorm%E3%81%A7%E3%81%AExdebug%E8%A8%AD%E5%AE%9A) 