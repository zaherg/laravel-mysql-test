# Example of using Laravel and Github Actions

This is just an example of how you can use GitHub Actions to test your laravel application

## Using Actions Services:

You can think of the services as the docker images that you want to use with your project, like: MySQL, Redis .. etc.

From GitHub Actions Docs:

> Additional containers to host services for a job in a workflow. These are useful for creating databases or cache 
> services like redis. The runner on the virtual machine will automatically create a network and manage the life cycle 
> of the service containers.
> 
> When you use a service container for a job or your step uses container actions, you don't need to set port information
> to access the service. Docker automatically exposes all ports between > containers on the same network.
> 
> When both the job and the action run in a container, you can directly reference the container by its hostname. The 
> hostname is automatically mapped to the service name.
> 
> When a step does not use a container action, you must access the service using localhost and bind the ports.


In the following example we create two services for MySQL and redis. GitHub selects an open port on the virtual host to bind 
the redis default port to. GitHub sets the bound host port in the ${{ job.services.<service_name>.ports[<port>] }} 
job context. For example, the redis port will be set in the ${{ job.services.redis.ports['6379'] }} environment variable.

```yaml
    services:
      mysql:
        image: mysql:5.7
        env:
          MYSQL_ALLOW_EMPTY_PASSWORD: false
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: laravel
        ports:
          - 3306/tcp
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3
      redis:
        image: redis
        ports:
          - 6379/tcp
        options: --health-cmd="redis-cli ping" --health-interval=10s --health-timeout=5s --health-retries=3
```

So in the job we can use this value like:

```yaml
      - name: Run PHPUnit
        run: php vendor/bin/phpunit
        env:
          DB_PORT: ${{ job.services.mysql.ports[3306] }}
          REDIS_PORT: ${{ job.services.redis.ports[6379] }}  
```