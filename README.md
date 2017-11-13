# How To Setup Ruby on Rails with Postgres

## 1. Init project
在本機端，我們無需安裝任何的 ruby 環境，我們只需要透過 docker hub 上提供的image，就可以幫助我們產生專案 source code

``` 
$ docker run --rm -v $(pwd):/usr/srv/app -w /usr/srv/app rails rails new . --skip-bundle
$ docker run --rm -v $(pwd):/usr/src/app -w /usr/src/app ruby:2.3.1 bundle lock

# option
$ docker run --rm -v $(pwd):/usr/src/app -w /usr/src/app ruby:2.3.1 bundle lock --update
```

## 2. Dockerfile
我們需要為這個專案添加一個專屬的 `Dockerfile`，執行安裝 `Gemfile` 內相依套件，並在最後啟動 `rails server`
```
FROM ruby:2.3.3
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
ADD Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
ADD . /myapp
CMD bundle exec rails s -p 3000 -b 0.0.0.0
```

## 3. Installing Requirements & Setup configuration
Gemfile
```
# Add gem "pg" into Gemfile 
# Add gem "rspec-rails" into Gemfile

gem "pg"
group :development, :test do
  gem 'rspec-rails', '~> 3.6'
end
```
Once configured, your database.yml should contain something like this:
```
default: &default
  adapter: postgresql
  encoding: unicode
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: helloworld
  username: develop
  password: password1
  host: helloworld_postgres_1

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: helloworld
  username: test
  password: password1
  host: helloworld_postgres_1


production:
  <<: *default
  database: helloworld
  username: prod
  password: password1
  host: helloworld_postgres_1
```

## 4. docker-compose.yml
```
version: '3.3'
services:
  postgres:
    image: postgres
    volumes:
      - postgres:/var/lib/postgresql/data
  web:
    build: .
    command: bundle exec rails s -p 8888 -b '0.0.0.0'
    volumes:
      - .:/myapp
    ports:
      - "3000:8888"
    depends_on:
      - postgres
volumes:
  postgres:
```

### 5. Setting Up Postgres
```
$ docker exec -it helloworld_postgres_1 psql -U postgres
postgres=# \?
\l[+]   [PATTERN]      list databases
\du[S+] [PATTERN]      list roles
\q                     quit psql
postgres=# create role develop with superuser login password 'password1';
postgres=# create role test with superuser login password 'password1';
postgres=# create role prod with superuser login password 'password1';
```
### 6. Rails command
```
$ docker exec -it helloworld_web_1 rails generate controller home index
$ docker exec -it helloworld_web_1 rake db:setup
$ docker exec -it helloworld_web_1 rails g scaffold Post title:string body:text
$ docker exec -it helloworld_web_1 rake db:migrate
$ docker exec -it helloworld_web_1 rails generate rspec:install
$ docker exec -it helloworld_web_1 bundle exec rspec
```

### Build & Run
- docker-compose build & docker-compose up -d

## Q&A
- [Difference between links and depends_on in docker_compose.yml](https://stackoverflow.com/questions/35832095/difference-between-links-and-depends-on-in-docker-compose-yml)
