# CLARITY docker-duplicity 

## Description

[duplicity](http://duplicity.nongnu.org/) is backup tool that is used by [CLARITY](https://clarity-h2020.eu/)'s Climate Services Information System ([CSIS](https://github.com/clarity-h2020/csis/)) on [CSIS Development System](https://github.com/clarity-h2020/csis#csis-development-system) and [CSIS Production System](https://github.com/clarity-h2020/csis#csis-production-system) virtual servers to perform incremental backups. 

## Implementation 

This is a fork of [wernight/docker-duplicity](https://github.com/wernight/docker-duplicity.git) (which we call *upstream* in the
following). We created this repo when *Duplicity* due to Python 2 deprecation moved to Python 3 but *upstream* did not follow yet. The according changes are commented in the subfolders' Dockerfiles and we changed all links in this README to our own version.

## Deployment

duplicity is available at `/docker/999-duplicity` on both the development and production server. It is not deployed directly as docker container, so it does not run in the background. It is intended to be started **on demand** to manually create [additional](https://github.com/clarity-h2020/docker-drupal/#backups) incremental backups before important changes to the system are made. 

### Configuration

#### docker-compose

The on-demand duplicity container is  managed via the customised [docker-compose.yml](https://github.com/clarity-h2020/docker-duplicity/blob/csis.ait.ac.at/docker-compose.yml). The actual configuration is maintained in an `.env` while [.env.sample](https://github.com/clarity-h2020/docker-duplicity/blob/csis.ait.ac.at/.env.sample) can be used as blueprint of this configuration file. Variables in this file will be substituted into [docker-compose.yml](https://github.com/clarity-h2020/docker-duplicity/blob/csis.ait.ac.at/docker-compose.yml) and [restore.yml](https://github.com/clarity-h2020/docker-duplicity/blob/csis.ait.ac.at/restore.yml). 

#### backed-up directories

Two separate backup configurations, [one](https://github.com/clarity-h2020/docker-duplicity/tree/csis-dev.ait.ac.at) for the development system and [another one](https://github.com/clarity-h2020/docker-duplicity/tree/csis.ait.ac.at) for the production system are maintained in two separate branches. The actual configuration consist of the `filelist.txt` file, that specifies which directories are backed-up by duplicity. 

The [filelist.txt](https://github.com/clarity-h2020/docker-duplicity/blob/csis-dev.ait.ac.at/filelist.txt) for [csis-dev.ait.ac.at](https://github.com/clarity-h2020/docker-duplicity/tree/csis-dev.ait.ac.at) looks for example like:

```txt
/data/docker/010-ngnix-proxy
/data/docker/000-hoster
/data/docker/020-pgadmin4/.env
/data/docker/100-csis
/data/docker/400-geonode/.env
/data/docker/400-geonode/*.env
/data/docker/500-ckan/.env
/data/docker/600-geoserver
/data/docker-volumes/ckan_config
/data/docker-volumes/ckan_home
/data/docker-volumes/ckan_pg-data
/data/docker-volumes/ckan_storage
/data/docker-volumes/pgadmin4_data
- **
```

Don't get confused by `/data` as it refers to the directory structure **inside of** the duplicity container. Refer to the **bind-mount** [volume declaration](https://github.com/clarity-h2020/docker-duplicity/blob/csis-dev.ait.ac.at/docker-compose.yml#L27) configuration in [docker-compose.yml](https://github.com/clarity-h2020/docker-duplicity/blob/csis-dev.ait.ac.at/docker-compose.yml#L27) to see how this maps to the host file system:

```yaml
services:
  duplicity
    volumes:
        - type: bind
          source: /docker
          target: /data/docker
        - type: bind
          source: /var/lib/docker/volumes
          target: /data/docker-volumes
```

## Usage

### Backup

Before creating the backup, it is advisable to stop any database containers to avoid database corruption in case of locked database files. For the CSIS containers ([docker-drupal](https://github.com/clarity-h2020/docker-drupal/issues)) this can .e.g done with:

```sh
cd /docker/100-csis
docker-compose stop
```

Creating an incremental backup is straightforward:

```sh
cd /docker/999-duplicity/
sudo docker-compose up
```

The typical output should look like:

> Starting duplicity-backup ... done 
> Attaching to duplicity-backup
> duplicity-backup | gpg: WARNING: unsafe permissions on homedir '/home/duplicity/.gnupg' 
> duplicity-backup | Reading globbing filelist /home/duplicity/filelist.txt
> duplicity-backup | Local and Remote metadata are synchronized, no sync needed. 
> duplicity-backup | Last full backup date: Thu Jul  2 07:26:11 2020 
> duplicity-backup | --------------[ Backup Statistics ]-------------- 
> duplicity-backup | StartTime 1596721659.90 (Thu Aug  6 13:47:39 2020) 
> duplicity-backup | EndTime 1596721819.03 (Thu Aug  6 13:50:19 2020) 
> duplicity-backup | ElapsedTime 159.13 (2 minutes 39.13 seconds) 
> duplicity-backup | SourceFiles 241798 
> duplicity-backup | SourceFileSize 7280718561 (6.78 GB) 
> duplicity-backup | NewFiles 6077 
> duplicity-backup | NewFileSize 326757996 (312 MB) 
> duplicity-backup | DeletedFiles 907 
> duplicity-backup | ChangedFiles 808
> duplicity-backup | ChangedFileSize 201415072 (192 MB) 
> duplicity-backup | ChangedDeltaSize 0 (0 bytes)
> duplicity-backup | DeltaEntries 7792
> duplicity-backup | RawDeltaSize 357169539 (341 MB)
> duplicity-backup | TotalDestinationSizeChange 242665597 (231 MB)
> duplicity-backup | Errors 0
> duplicity-backup | -------------------------------------------------
> duplicity-backup |
> duplicity-backup exited with code 0

### Restore

**The latest** backup-set can be easily restored with with 

```sh
cd /docker/999-duplicity/
docker-compose up -f restore.yml
``` 
The restored directories can then be found in the `/docker/999-duplicity/restore` directory. For other restore options please refer to the [duplicity documentation](http://duplicity.nongnu.org/docs.html).

# About wernight/docker-duplicity

## What is Duplicity?

**[duplicity](http://duplicity.nongnu.org/)** backup tool.

Features of this Docker image:

  * **Small**: Built using [alpine](https://hub.docker.com/_/alpine/).
  * **Simple**: Most common cases are explained below and require minimal setup.
  * **Secure**: Runs non-root by default (use randomly chosen UID `1896`), and meant to run as any user.


## Usage

For the general command-line syntax, do:

    $ docker run --rm bludoc/duplicity

In general you...

  * Must mount what you want to backup or where you want to restore a backup.
  * Should mount `/home/duplicity/.cache/duplicity` as writable somewhere (if not cached, [duplicity will have to recreate it from the remote repository which may require decrypting the backup contents](http://duplicity.nongnu.org/duplicity.1.html#sect5)). Note it may be quite large and contains metadata info about files you've backed up in clear text.
  * Should mount `/home/duplicity/.gnupg` as writable somewhere (that directory is used to validate incremental backups and shouldn't be necessary to restore your backup if you follows steps below).
  * Should specify duplicity flag `--allow-source-mismatch` because Docker has a random host for each container.
  * Could set environment variable `PASSPHRASE`, unless you want to type it manually in the prompt (remember then to add `-it`).
  * May have to mount a few other files for authentication (see examples below).


Example of commands you may want to run **periodically to back up** with good clean-up/maintenance (see below for various storage options):

     $ docker run --rm ... bludoc/duplicity --full-if-older-than=6M source_directory
      target_url
     $ docker run --rm ... bludoc/duplicity remove-older-than 6M --force target_url
     $ docker run --rm ... bludoc/duplicity cleanup --force target_url

This would do:

 1. A full backup every 6 months so that restoration is a lot faster and for cleanup to work,
    and incremental backups the rest of the time.
 2. Delete backups older than 6 months (doesn't break incremental backups).
 3. Delete files from failed sessions (if any).


### Backup to **Google Cloud Storage** example

**[Google Cloud Storage](https://cloud.google.com/storage/)** *nearline* [costs about $0.01/GB/Month](https://cloud.google.com/storage/pricing).

**Set up**:

 1. [Sign up, create an empty project, enable billing, and create a *bucket*](https://cloud.google.com/storage/docs/getting-started-console)
 2. Under ["Storage" section > "Settings"](https://console.cloud.google.com/project/_/storage/settings) > "Interoperability" tab > click "Enable interoperable access" and then "Create a new key" button and note both *Access Key*	and *Secret*. Also note your *Project Number* (aka project ID, it's a number like 1233457890).
 3. Run [gcloud's `gsutil config -a`](https://cloud.google.com/storage/docs/getting-started-gsutil) to generate the `~/.boto` configuration file and give it all these info (alternatively you should be able to set environment variable `GS_ACCESS_KEY_ID` and `GS_SECRET_ACCESS_KEY` however in my tries I didn't see where to set your project ID).
 4. You should now have a `~/.boto` looking like:

        [Credentials]
        gs_access_key_id = MYGOOGLEACCESSKEY
        gs_secret_access_key = SomeVeryLongAccessKeyXXXXXXXX
    
        [GSUtil]
        default_project_id = 1233457890

Now you're ready to perform a **backup**:

    $ docker run --rm --user $UID \
          -e PASSPHRASE=P4ssw0rd \
          -v $PWD/.cache:/home/duplicity/.cache/duplicity \
          -v $PWD/.gnupg:/home/duplicity/.gnupg \
          -v ~/.boto:/home/duplicity/.boto:ro \
          -v /:/data:ro \
          bludoc/duplicity \
          duplicity --full-if-older-than=6M --allow-source-mismatch /data gs://my-bucket-name/some_dir

To **restore**, you'll need:

  * Keep `.boto` or regenerate it to access your Google Cloud Storage.
  * The `PASSPHRASE` you've used.

Example:

    $ docker run --rm --user $UID \
          -e PASSPHRASE=P4ssw0rd \
          -v ~/.boto:/home/duplicity/.boto:ro \
          -v /:/data:ro \
          bludoc/duplicity \
          duplicity restore gs://my-bucket-name/some_dir /data
          
See also the [note on Google Cloud Storage](http://duplicity.nongnu.org/duplicity.1.html#sect15).


### Backup to **Google Drive** example

**[Google Drive](https://drive.google.com/)** offers [15GB for free](https://support.google.com/drive/answer/2375123).

**Set up**:

 1. Follow notes [on Pydrive Backend](http://duplicity.nongnu.org/duplicity.1.html#sect20) to generate a P12 credential file (call it `pydriveprivatekey.p12`) and note also the associated service account email generated (e.g. `duplicity@developer.gserviceaccount.com`).
 2. Convert P12 to PEM:

        $ docker run --rm -i --user $UID \
              -v $PWD/pydriveprivatekey.p12:/pydriveprivatekey.p12:ro \
              bludoc/duplicity \
              openssl pkcs12 -in /pydriveprivatekey.p12 -nodes -nocerts >pydriveprivatekey.pem
        Enter Import Password: notasecret

Now you're ready to perform a **backup**:

    $ docker run --rm --user $UID \
          -e PASSPHRASE=P4ssw0rd \
          -e GOOGLE_DRIVE_ACCOUNT_KEY=$(cat pydriveprivatekey.pem) \
          -v $PWD/.cache:/home/duplicity/.cache/duplicity \
          -v $PWD/.gnupg:/home/duplicity/.gnupg \
          -v /:/data:ro \
          bludoc/duplicity \
          duplicity --full-if-older-than=6M --allow-source-mismatch /data pydrive://duplicity@developer.gserviceaccount.com/some_dir

To **restore**, you'll need:

  * Regenerate a PEM file (or keep it somewhere).
  * The `PASSPHRASE` you've used.

### Backup via **rsync** example

Supposing you've an **SSH** access to some machine, you can:

    $ docker run --rm -it --user root \
          -e PASSPHRASE=P4ssw0rd \
          -v $PWD/.cache:/home/duplicity/.cache/duplicity \
          -v $PWD/.gnupg:/home/duplicity/.gnupg \
          -v ~/.ssh/id_rsa:/id_rsa:ro \
          -v ~/.ssh/known_hosts:/etc/ssh/ssh_known_hosts:ro \
          -v /:/data:ro \
          bludoc/duplicity \
          duplicity --full-if-older-than=6M --allow-source-mismatch \
          --rsync-options='-e "ssh -i /id_rsa"' \
          /data rsync://user@example.com/some_dir

Note: We're running here as `root` to have access to `~/.ssh` and also because ssh does not
allow to use a random (non-locally existing) UID. To make it safer, you can copy your `~/.ssh`
and `chown 1896` it (that is `duplicity` UID within the container). If you know a another way to avoid
the "No user exists for uid" check, please let me know.


## Alias

Here is a simple alias that should work in most cases:

    $ alias duplicity='docker run --rm --user=root -v ~/.ssh/id_rsa:/home/duplicity/.ssh/id_rsa:ro -v ~/.boto:/home/duplicity/.boto:ro -v ~/.gnupg:/home/duplicity/.gnupg -v /:/mnt:ro -e PASSPHRASE=$PASSPHRASE bludoc/duplicity duplicity $@'

Now you should be able to run duplicity almost as if it were installed, example:

    $ PASSPHRASE=123456 duplicity --progress /mnt rsync://user@example.com/some_dir


## See also

  * [duplicity man](http://duplicity.nongnu.org/duplicity.1.html) page
  * [duplicity back-up how-to - Ubuntu](https://help.ubuntu.com/community/DuplicityBackupHowto)
  * [How To Use Duplicity with GPG to Securely Automate Backups on Ubuntu | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-use-duplicity-with-gpg-to-securely-automate-backups-on-ubuntu)


## Feedback

Report issues/questions/feature requests on *upstream*'s [GitHub Issues
](https://github.com/wernight/docker-duplicity/issues).
