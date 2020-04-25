# dxpr/repository

`satis` is used to build composer repositories.

## Rules

- We have different repository end-points for different Drupal major version matching the branch name.

- `7` branch name  => packages.dxpr.com/7
- `8` branch name  => packages.dxpr.com/8
- `9` branch name  => packages.dxpr.com/9
- `10` branch name => packages.dxpr.com/10
- ...

## How to use: setting up this repository in your projects

- Add this Composer repository to your project's `composer.json file`, then you can require these
private packages just like you would with one from Packagist.

```json
{
    "repositories": [{
        "type": "composer",
        "url": "https://packages.dxpr.com/8"
    }]
}
```

- How to authenticate:
  //TODO(hoatle): this is not yet working, the repo is still public for now


## Set up the build

- Make sure `docker` and `docker-compose` is available.

- Generate a ssh key, don't put any passphrase:

```
$ mkdir .ssh
$ ssh-keygen -t rsa -b 4096 -C "repository@dxpr.com" -f .ssh/repository_rsa
```

- Copy the public key to your account (or any user account) that has access to the repos defined on the
  `satis.json` file. It's a best practice that you should creat a bot account for your organization
  with read-only access which owns this generated key pair.


- Verify that it works, for example:

```
$ docker-compose run build bash -c "ssh -oStrictHostKeyChecking=accept-new -T -ai ~/.ssh/repository_rsa git@github.com"
Warning: Permanently added 'github.com,140.82.113.4' (RSA) to the list of known hosts.
Hi hoatle! You've successfully authenticated, but GitHub does not provide shell access.
```

- Create the github personal access token (PAT):

  + Head to https://github.com/settings/tokens/new?scopes=repo&description=dxpr-repository
    to retrieve a token.

  + Create the `.composer/auth.json` config and then execute:

```bash
$ export GH_PAT=<fill_in_the_created_PAT_here>
$ mkdir -p .composer
$ cat <<EOF >.composer/auth.json
{
  "github-oauth": {
      "github.com": "$GH_PAT"
  }
}
EOF
```

## How to run the build


- Build it with `docker-compose`:

```bash
$ docker-compose up
# you see see the following similar output:
Creating repository_build_1 ... done
Attaching to repository_build_1
build_1  | No explicit requires defined, enabling require-all
build_1  | Scanning packages
build_1  | Selected dxpr/builder (1.4.4)
build_1  | Selected dxpr/builder (dev-master)
build_1  | Selected dxpr/builder (1.4.x-dev)
build_1  | Selected dxpr/theme (dev-master)
build_1  | Creating local downloads in 'web/8/dist'
build_1  | Dumping package 'dxpr/builder' in version '1.4.4'.
build_1  | Skipping 'dxpr/theme dev-master' (is dev)
build_1  | Skipping 'dxpr/builder 1.4.x-dev' (is dev)
build_1  | Skipping 'dxpr/builder dev-master' (is dev)
build_1  | Wrote packages to web/8/include/all$22be02e69b7f25122547f4569f4a2b6599bb50f7.json
build_1  | Writing packages.json
build_1  | Pruning include directories
build_1  | Deleted web/8/include/all$3d577f6a030b0943d0e1427b27eb210308cb5869.json
build_1  | Writing web view
repository_build_1 exited with code 0
```

- You should see the fimilar generated output below:

```bash
$ tree web/
web/
└── 8
    ├── dist
    │   └── dxpr
    │       └── builder
    │           └── dxpr-builder-1.4.4-4540e5.zip
    ├── include
    │   └── all$d5676d6290c66ec7f0cb30f8ef69e58444936422.json
    ├── index.html
    └── packages.json

5 directories, 4 files
```

- To clean up:

```bash
$ rm -rf .composer/cache
$ rm -rf web
$ docker-compose down -v
```

## How to publish on DigitalOcean Spaces

- You can publish to DigitalOcean Spaces or AWS S3, they should work the same.

- Follow:
  + https://www.digitalocean.com/docs/spaces/resources/s3cmd/
  + https://www.digitalocean.com/docs/spaces/resources/s3cmd-usage/

- Then:

```bash
$ export AWS_ACCESS_KEY_ID=<fill_yours>
$ export AWS_SECRET_ACCESS_KEY=<fill_yours>
$ export BUCKET=<fill_yours>
$ rm -rf web # clean up
$ docker-compose up # build
$ s3cmd sync -r --delete-removed web/8 s3://$BUCKET/ # sync
$ s3cmd setacl s3://$BUCKET --acl-public --recursive # set acl if needed
```



## How to publish on AWS S3

//TODO(hoatle): add this

