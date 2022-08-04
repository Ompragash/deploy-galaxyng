###  Environment
----------------
Tested on:
  - AlmaLinux 8.6, ~=9
  - Fedora 36
  - RHEL & CentOS ~=8, ~=9

### Requirements
----------------
- Podman or Docker
- Git

### Deployment Instruction
--------------------------
Install required OS packages
```bash
dnf install -y git podman
```

Clone [deploy-galaxyng](https://github.com/Ompragash/deploy-galaxyng) git repo
```bash
git clone https://github.com/Ompragash/deploy-galaxyng.git
cd deploy-galaxyng
```

Pull `pulp/pulp-galaxy-ng` image. In this guide, we'll be using _podman_ but if you are familar with _docker_ then replace _podman -> docker_ and follow along:
```bash
podman pull pulp/pulp-galaxy-ng
```

Create required files and directories for use of `pulp-galaxy-ng` container
```bash
mkdir galaxyng && cd galaxyng
mkdir settings pulp_storage pgsql containers

echo "CONTENT_ORIGIN='http://$(hostname):8080'
ANSIBLE_API_HOSTNAME='http://$(hostname):8080'
ANSIBLE_CONTENT_HOSTNAME='http://$(hostname):8080/pulp/content'"  >> settings/settings.py
```

> _**Note:** To use a different port (eg: 9090) then replace 8080 in the echo command._

There are two ways to start the container.
- With SELinux
    ```bash
    podman run --detach \
             --publish 8080:80 \
             --name galaxy_ng \
             --volume ./settings:/etc/pulp:Z \
             --volume ./pulp_storage:/var/lib/pulp:Z \
             --volume ./pgsql:/var/lib/pgsql:Z \
             --volume ./containers:/var/lib/containers:Z \
             --device /dev/fuse \
             pulp/pulp-galaxy-ng
    ```
- Without SELinux
    ```bash
    podman run --detach \
             --publish 8080:80 \
             --name galaxy_ng \
             --volume ./settings:/etc/pulp \
             --volume ./pulp_storage:/var/lib/pulp \
             --volume ./pgsql:/var/lib/pgsql \
             --volume ./containers:/var/lib/containers \
             --device /dev/fuse \
             pulp/pulp-galaxy-ng
    ```

Inject initial data
```bash
cat initial_data.json | podman exec -i galaxy_ng bash -c "cat > /tmp/initial_data.json"

podman exec galaxy_ng bash -c "/usr/local/bin/pulpcore-manager loaddata /tmp/initial_data.json"
```

> _**NOTE:** The initial_data.json adds `galaxy-org` group and `admin` user._

To reset `admin` password run the below command
```bash
podman exec -it galaxy_ng bash -c 'pulpcore-manager reset-admin-password'
```

All set!🎉 Now access the Galaxy NG UI with _http://HOST_FQDN:PORT_ where HOST_FQDN is `localhost` in this case and the PORT is `8080`.