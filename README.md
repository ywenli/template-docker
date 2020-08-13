# template-docker

This is a template file for building a Ruby on Rails environment.
 
The environment to be built is as follows.

- ruby 2.5.3
- rails 5.2.4.2
- MySQL 8.0.13

You can have them change their versions as needed.

## Instruction

Follow the steps below:

### 1. Dockerfile

Create a new Dockerfile and put the following in the Dockerfile:

```Docker:Dockerfile
FROM ruby:2.5.3                         
#rubyのバージョン指定
RUN gem install rails
#gemのインストール
RUN apt-get update && apt-get install -y nodejs mysql-client
#パッケージのアップデート　インストール

COPY Gemfile /Gemfile
COPY Gemfile.lock /Gemfile.lock
#ローカルのファイルをコンテナにコピー
RUN bundle install
```

### 2. docker-compose.yml

```
version: '3'
services:
  mysql:           #コンテナの名前(なんでもok)
    image: mysql:8.0.13
    command: --default-authentication-plugin=mysql_native_password
                   #MySQL -v8.0から認証方式が変更されたので、mysql -v5.0に合わせるプラグイン
    volumes:
      - "./mysql-data:/var/lib/mysql"
                   #ローカルのmysql-dateにデータを保存する（コンテナが消えてもデータが残る）
    environment:
      MYSQL_ROOT_PASSWORD: root



  app:               #コンテナの名前(なんでもok)
    build: .         #Dockefileを指定
    volumes:         #dockerコンテナ内のファイルをローカルで編集する為の記述
      - ".:/app"
      - gem_data:/usr/local/bundle
                     #gemの永続化
    ports:           #localhost:3000にアクセスすると、コンテナ内の3000ポートにアクセスする
      - "3000:3000"
    tty: true
    depends_on:
      - mysql        #mysqlコンテナにアクセスする
    working_dir: "/app" #コンテナに入った時に直接appディレクトリに入る

volumes:
  mysql-data:
  gem_data:
```

### 3. Gemfile

```
source 'https://rubygems.org'
gem 'rails', '~> 5.2.0'
```

### 4. Gemfile.lock

```touch Gemfile.lock```

### 5. Docker 起動

```docker-compose up -d```

### 6. コンテナに入る

```docker exec -it sample_app_1 /bin/bash```

### 7. rails new

```rails new . --force --database=mysql --skip-bundle```

### 8. サーバー立ち上げ

```rails s -b 0.0.0.0```


