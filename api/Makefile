# Include file with .env variables if exists
-include .env

# Define default values for variables
COMPOSE_FILE ?= docker-compose.yml
BASE_IMAGE_DOCKERFILE ?= ./.docker/prod/base/Dockerfile
IMAGE_REGISTRY ?= prod
IMAGE_TAG ?= latest

# Define your docker-compose version
# eg: 1 or 2
# 1 = docker-compose
# 2 = docker compose
DOCKER_COMPOSE_VERSION ?= 2
DOCKER_COMPOSE ?= docker-compose

# check docker-compose version
ifeq ($(DOCKER_COMPOSE_VERSION), 2)
	DOCKER_COMPOSE = docker compose
endif


#-----------------------------------------------------------
# Management
#-----------------------------------------------------------

# Create shared gateway network
gateway:
	docker network create gateway

# Init variables for development environment
env.dev:
	cp ./.env.dev ./.env

# Init variables for production environment
env.prod:
	cp ./.env.prod ./.env

# Build and restart containers
install: build.all up

# Start containers
up:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} up -d

# Stop containers
down:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} down --remove-orphans

# Build containers
build:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} build

# Build all containers
build.all: build.base build

# Build the base app image
build.base:
	docker build --file ${BASE_IMAGE_DOCKERFILE} --tag ${IMAGE_REGISTRY}/api-base:${IMAGE_TAG} .

# Show list of running containers
ps:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} ps

# Restart containers
restart:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} restart

# Reboot containers
reboot: down up

# View output from containers
logs:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} logs --tail 500

# Follow output from containers (short of 'follow logs')
fl:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} logs --tail 500 -f

# Prune stopped docker containers and dangling images
prune:
	docker system prune

#-----------------------------------------------------------
# Application
#-----------------------------------------------------------

# Enter the app container
app.bash:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app /bin/bash

# Restart the app container
restart.app:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} restart app

# Alias to restart the app container
ra: restart.app

# Run the tinker service
tinker:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app php artisan tinker

# Clear the app cache
cache.clear:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app php artisan cache:clear

# Migrate the database
db.migrate:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app php artisan migrate

# Alias to migrate the database
migrate: db.migrate

# Rollback the database
db.rollback:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app php artisan migrate:rollback

# Seed the database
db.seed:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app php artisan db:seed

# Fresh the database state
db.fresh:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app php artisan migrate:fresh

# Refresh the database
db.refresh: db.fresh db.seed

# Dump database into file (only for development environment) (TODO: replace file name with env variable)
db.dump:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec postgres pg_dump -U ${DB_USERNAME} -d ${DB_DATABASE} > ./.docker/postgres/dumps/dump.sql

# TODO: add command to import db dump

# Restart the queue process
queue.restart:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec queue php artisan queue:restart

# Install composer dependencies
composer.install:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app composer install

# Install composer dependencies from stopped containers
r.composer.install:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} run --rm --no-deps app composer install

# Alias to install composer dependencies
ci: composer.install

# Update composer dependencies
composer.update:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app composer update

# Update composer dependencies from stopped containers
r.composer.update:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} run --rm --no-deps app composer update

# Alias to update composer dependencies
cu: composer.update

# Show outdated composer dependencies
composer.outdated:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app composer outdated

# PHP composer autoload command
composer.autoload:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app composer dump-autoload

# Generate a symlink to the storage directory
storage.link:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app php artisan storage:link --relative

# Give permissions of the storage folder to the www-data
storage.perm:
	sudo chmod -R 755 storage
	sudo chown -R www-data:www-data storage

# Give permissions of the storage folder to the current user
storage.perm.me:
	sudo chmod -R 755 storage
	sudo chown -R "$(shell id -u):$(shell id -g)" storage

# Give files ownership to the current user
own.me:
	sudo chown -R "$(shell id -u):$(shell id -g)" .

# Reload the Octane workers
octane.reload:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app php artisan octane:reload

# Alias to reload the Octane workers
or: octane.reload

#-----------------------------------------------------------
# Testing (only for development environment)
#-----------------------------------------------------------

# Run phpunit tests (requires 'phpunit/phpunit' composer package)
test:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app ./vendor/bin/phpunit --order-by=defects --stop-on-defect

# Alias to run phpunit tests
t: test

# Run phpunit tests with the coverage mode (TODO: install PCOV or other lib)
coverage:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app ./vendor/bin/phpunit --coverage-html ./.coverage

# Run dusk tests (requires 'laravel/dusk' composer package)
dusk:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app php artisan dusk

# Generate code metrics (requires 'phpmetrics/phpmetrics' composer package)
metrics:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec app ./vendor/bin/phpmetrics --report-html=./.metrics api/app

#-----------------------------------------------------------
# Redis
#-----------------------------------------------------------

# Enter the redis container
redis:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec redis redis-cli

# Flush the redis state
redis.flush:
	${DOCKER_COMPOSE} -f ${COMPOSE_FILE} exec redis redis-cli FLUSHALL

#-----------------------------------------------------------
# Swarm
#-----------------------------------------------------------

# Deploy the stack
swarm.deploy:
	docker stack deploy --compose-file ${COMPOSE_FILE} api

# Remove/stop the stack
swarm.rm:
	docker stack rm api

# List of stack services
swarm.services:
	docker stack services api

# List the tasks in the stack
swarm.ps:
	docker stack ps api

# Init the Docker Swarm Leader node
swarm.init:
	docker swarm init

#-----------------------------------------------------------
# Danger zone
# Do not use these commands
#-----------------------------------------------------------

# Remove all app files and folders (leave only dockerized template)
danger.app.prune:
	sudo rm -rf \
		.idea \
		.vscode \
		.vscode \
		app \
		bootstrap \
		config \
		database \
		lang \
		public \
		resources \
		routes \
		storage \
		vendor \
		tests \
		.editorconfig \
		.env \
		.env.example \
		.gitattributes \
		.gitignore \
		.phpunit.result.cache \
		.styleci.yml \
		artisan \
		composer.json \
		composer.lock \
		package.json \
		phpunit.xml \
		README.md \
		webpack.mix.js
