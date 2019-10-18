# Manage AWS users, permissions and buckets

## General
Scripts for management of user, buckets and permissions for:
* restic backup user + buckets
* s3sync backup user + buckets
* traefik route53 user

### Requirements
```
* ansible (pip3 install ansible)
* boto3 (pip3 install boto)
```

## Backups, restic and s3sync

### Prepare
Edit the `backup_sets` file and add the backup clients under `[restic-clients]` or `[s3sync-clients]`. For example
```
[local]
127.0.0.1   ansible_connection=local

[restic-clients]
set-one
set-two

[s3sync-clients]
set-three

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
ansible-playbook -i backup_sets -e access=<aws access key> -e secret=<aws secret key> rotate_restic_keys.yml
ansible-playbook -i backup_sets -e access=<aws access key> -e secret=<aws secret key> rotate_s3sync_keys.yml
```
You can add `--limit set-one` to the command to just rotate keys for *set-one*


### Delete user and bucket (restic only)

Add single client into delete_sets, enter client name again during prompt.

Run
```
ansible-playbook -i delete_sets -e access=<aws access key> -e secret=<aws secret key> delete_user_and_bucket.yml
```

## traefik route53 
This creates and user with permissions to create DNS records in specified hosted domains, this can be used in Treafik configuration with route53 cert provider.

### Prepare

Edit file traefik_route53_sets  Example: 
```
[local]
127.0.0.1   ansible_connection=local

[traefik-clients]
ainature
analytics
```
Create for each traefik client a file in host_vars named <client>.yml , Below ainature.yml as example:
```
name: ainature
hostedzoneids:
  ainature.nl: Z1ZDXJ9S0ZVP6U
  ainature.eu: Z1UXY2QNPHVTVZ
```
The script only used the value of the hostedzoneids and the backupset name, not the name: in host_vars is uses in naming the created user. keys of hostedzoneids and name: in host_vars file are just for making the file easier to read and understand. 

### Create users: 
```
ansible-playbook -i traefik_route53_sets -e access=<aws access key> -e secret=<aws secret key> create_traefik_route53_user.yml 
```

### Rotate keys

Run
```
ansible-playbook -i backup_sets -e access=<aws access key> -e secret=<aws secret key> rotate_traefik_route53_keys.yml
```
You can add `--limit set-one` to the command to just rotate keys for *set-one*






