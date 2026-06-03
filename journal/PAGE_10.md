# PAGE 5

## time: 03.06.2026, 14:09 EET 

## Status

Today is a great day. I managed to integrate the newly designed Django models: `PostgreSQLInstance`, 
`PostgreSQLDatabase`, `PostgreSQLBackup`, `PostgreSQLUser` with the Ansible logic.

What I managed to do is this workflow: 

1) The user requests a new VM (this was already implemented).
2) After the VM is created, we perform an API POST request to insert the PostgreSQL instance
3) After we managed to insert the PostgreSQL instance, we insert the `postgres` user
4) In the end, we insert each database (postgres, template0 and template1) contained by the PostgreSQL instance.

I tried to create a diagram of the logic mentioned above. If you are interested, please have a look over here: 
[API_Logic_When_Provisioning_New_VM](../docs/API_Logic_When_Provisioning_New_VM.png)

This logic basically explains how we update the Django API database with all the data we obtain after the provisioning 
is done.  

Some interesting things/cases I stumbled upon today are: 

1) The error of referencing the same instance (from different VM) when trying to insert a specific database:

```json
fatal: [test2 -> localhost]: FAILED! => {
    "allow": "GET, POST, HEAD, OPTIONS",
    "changed": false, 
    "content_length": "77",
    "content_type": "application/json", 
    "cross_origin_opener_policy": "same-origin", 
    "date": "Wed, 03 Jun 2026 10:41:05 GMT", 
    "elapsed": 0, 
    "json": {
        "non_field_errors": [
            "The fields db_name, instance must make a unique set."
        ]
    }, 
    "msg": "Status code was 400 and not [201]: HTTP Error 400: Bad Request", 
    "redirected": false, 
    "referrer_policy": "same-origin", 
    "server": "WSGIServer/0.2 CPython/3.9.25", 
    "status": 400, 
    "url": "http://127.0.0.1:9001/api/dbs/", 
    "vary": "Accept, Cookie", 
    "x_content_type_options": "nosniff", 
    "x_frame_options": "DENY"
}
```

By mistakenly trying to reference the same instance, the one from the test1 VM, when trying to add a DB from the test2 
VM, the unique constraint `('db_name', 'instance')` of the `PostgreSQLDatabase` model is triggered. 

To solve that, we performed the API GET request to obtain all the PostgreSQL instances and filtered the result locally 
using the `vm_id` value. Check how we create the `fetched_instance_url_global` variable.

```ansible
- name: "Insert existing DBs"
  vars:
    vm_id: "{{ vm_id }}"
    fetched_instance_url_global: "{{ (instance_get_output.json.results | selectattr('vm', 'search', '/' ~ vm_id ~ '/') | first).url }}"
  ansible.builtin.include_role:
    name: django_api_client
    tasks_from: insert_psql_dbs_day_0
  loop: "{{ instance_databases_output.stdout_lines }}"
```
