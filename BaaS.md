# Backup as a Service

The Backup process entails a collection of ansible scripts that manipulate the servers to leave them in a “blank” state suitable for performing a consistent backup, a specific tool that is able to backup a complete Linux machine into an ISO image that later can be used to restore and an openstack CLI integration that allows the operator to perform the actions in a very easy way.

This backup automation has a lot of things built in and it is very helpful, but it still lacks some degree of integration. To solve that, we are going to implement what we call backup as a service. This service will be a piece of software that will run on a container in the openstack’s overcloud, exposing an API that can be accessed by third parties to trigger or program backups, to list all the history of backups that are stored in the system and allow to trigger restorations or delete old backups. It will even be capable of accepting backup programming and programmed cleanups.

As Openstack on Openshift deployed via an operator seems to be the future of Openstack, my proposal is to write this Service in Golang, which is much better aligned with the Openshift/Kubernetes/containers environment than Openstack’s traditional Python.

## BACKUP

The basic API that will be implemented in the service is a kind of a CRD (Create, Read, Delete) to manage the backups. The API will be restful and versioned, so five basic call URLs will look like this:

```
# Return the complete list of backups
GET https://ip:port/v1/backup

# Return one backup
GET https://ip:port/v1/backup/{id}

# Return last backup
GET https://ip:port/v1/backup/last

# Create a new backup (the overcloud will create 3 different backups that can be restored independently, but they **cannot** be backuped individually)
POST https://ip:port/v1/backup/{undercloud|overcloud}?stopall=[**no**|yes]

# Delete a backup
DELETE https://ip:port/v1/backup/{id}
```

We have deliberatedly ignored the "update" command because backups should be immutable, and once it has been taken, it can be deleted or restored, but never changed.

As the bakups are stored remotely, in a remote storage that we cannot take in account if it will be available at all times or what type of storage will be, we should not rely on the mere existence of files on a filesystem to retrieve all the information regarding the backups. For tracing that we will use Openstack's MySQL database in the undercloud. The service will be executing in the undercloud, from where it will use ansible to take backups of the whole control plane.

The backups table of the database should contain at least, the following data:

| ID | NAME | LOCATION_URL | CREATION_DATE |

To document the API and be able to expose it to third parties, we will use the OpenAPI specification (formerly known as Swagger). The API documentation will be embedded on the service binary.

## RESTORE

The second leg of the backup and restore procedure is the restore. The idea is to use the service we are implementing to give the user capabilities over the restore.

As a PXE-based restoration is on the way. Once that gets done, the restoration shall not need manual restoration. An API call of the likes:

```
# Restores the backup {id} to the {nodeid}.
POST https://ip:port/v1/restore/{id}/{nodeid}
```

The *nodeid* is obtained using openstack CLI with undercloud (.stackrc) activated.

The invoked procedure is not yet implemented, but, as with backup, it could be contained within an ansible playbook.

Of course, the service should keep an historical restore table that could be queried via API operations:

```
# Return the complete list of restores
GET https://ip:port/v1/restore

# Return one restore
GET https://ip:port/v1/restore/{id}

# Return last restore
GET https://ip:port/v1/restore/last
```

