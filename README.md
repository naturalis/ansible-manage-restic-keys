# Manage keys for Restic

### Requirements
```
* ansible (pip3 install ansible)
* boto (pip3 install boto)
```

### Prepare
Edit the `backup_sets` file and add the backup clients under `[restic-clients]`. For example
```
[local]
127.0.0.1   ansible_connection=local

[restic-clients]
set-one
set-two
```
Make sure that the name is *dns compatable*

### Create user and buckets

Run
```
ansible-playbook -i backup_sets -e access=<aws access key> -e secret=<aws secret key> create_user_and_bucket.yml
```
This will create a user `restic-client-<name>` and a bucket `restic-<name>`. The `access_key_id` 
and `secret_access_key` will be printed to stdout. This is only available when the user is created. 

```
ansible-playbook -i backup_sets -e access=<aws access key> -e secret=<aws secret key> create_s3sync_user_and_bucket.yml
```
This will create a user `s3sync-client-<name>` and a bucket `s3sync-<name>`. The `access_key_id` 
and `secret_access_key` will be printed to stdout. This is only available when the user is created. The bucket will be created with versioning enabled.

After creation it may be usefull to set a lifecycle rule for the noncurrent object, noncurrent object are possible when versioning is enabled and objects are deleted or overwritten. Currently there is no support within ansible for creating this lifecycle rule so a awscli command should be used from within this repository.
```
aws s3api put-bucket-lifecycle --bucket s3sync-<bucket name> --lifecycle-configuration file://s3sync-lifecycle.json
```


### Rotate keys

Run
```
ansible-playbook -i backup_sets -e access=<aws access key> -e secret=<aws secret key> create_user_and_bucket.yml
```
You can add `--limit set-one` to the command to just rotate keys for *set-one*


### Delete user and bucket

Run
```
ansible-playbook -i backup_sets -e access=<aws access key> -e secret=<aws secret key> delete_user_and_bucket.yml
```
You can add `--limit set-one` to the command to just delete  *set-one*




