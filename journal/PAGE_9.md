# PAGE 5

## time: 20.05.2026, 16:17 EET 

## Status

Today I managed to customize the procedure of API usage to create a custom PostgreSQL provisioned VM. 
By "custom" I mean that the user can give a name for the VM he performs a POST request for. Using only that name, 
he can perform the POST request and from there a task will be created and saved into the Redis database and then 
performed by the Celery worker. 

About the provisioning procedure, I modified a bit the Ansible playbook so that after the new VM is created, it 
performs a PATCH request (using the ansible.builtin.uri module) and updates the IP address saved inside the SQLite3 
database. 

## Useful commands used

These are some commands used in order to test/use the project so far: 

In case you modify something structural inside the SQLite database (I mean the models here), use: 
```commandline
migrate-pg-api makemigrations
migrate-pg-api migrate
```

At the moment this entrypoint (migrate-pg-api) inside the django_rest_api package can be used with a general purpose (
I know, it is a problem... I will fix it later)

So, in case you want to flush the SQLite database, you can use: 

```commandline
migrate-pg-api flush
```

This will erase everything you have inside your database, including users etc. etc. 

Ok, now that we talk about structure, I want to move on to other important aspect. That is related to the Celery worker.
Everytime you create a new release or simply update your Python packages, you need to restart the Celery process. Why 
we need to do that? Because Celery loads the whole Python source code in RAM when the process is started. The code 
remains there until the Celery is stopped. In order to apply our changes, we need to stop Celery: 

```commandline
CTRL + C 
sudo pkill -9 -f celery
```

And then we have to start it back:

```commandline
celery -A provisioner_api.tasks worker --loglevel=info
```

In comparison with the Celery, Django does not need restart.

Moving on...to start the Django server, you can use: 

```commandline
run-pg-api 
```

This is another entrypoint defined by us inside the setup.py of the django_rest_api package. 

Other entrypoints defined for the django_rest_api packages would be: 

```commandline
pg-api-create-superuser -u <username> -e <email> -p <password>
pg-api-create-user -u <username> - e <email> -p <password>
```

I know, the password should not be in plain text, but for an MVP is not important. Later I will deal with this problem.

## TO-DO for tomorrow

- Start documenting the project (there are some important aspects that I want to document related to Django).
  - How I managed to update the IP address of the newly created VM. The logic is interesting.
  - Style a bit the code.
  - Journal the features, limitations and stuff like that
  - Create a TO-DO list for the future (I really want to implement some CI/CD tests and linting) 
- Create a demo video
- Create a LinkedIn article where to showcase the achievement
