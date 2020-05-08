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

- Recommend way on how users should authenticate with provided JWT token before `composer install`:

```bash
$ composer config --global bearer.<domain> <token>
```

For example:

```bash
$ composer config --global bearer.packages.dxpr.com/8 eyJraWQiOiIxWlowN2FMVF9IOGVnc3JSU1VvMmVkc2dzbUtNVTJCRzBhSmZGZFNiWF9VIiwiYWxnIjoiUFMyNTYifQ.eyJpc3MiOiJodHRwczovL2R4cHIuY29tIiwic3ViIjoidXNlci1pZCIsImF1ZCI6Imh0dHBzOi8vcGFja2FnZXMuZHhwci5jb20iLCJzY29wZSI6Ijg6ZHhwci9keHByX2J1aWxkZXI6KiA5OmR4cHIvZHhwcl9idWlsZGVyOioiLCJpYXQiOjE1ODg2MTMwNzB9.YPrGULY4TUm8Ck6CXU1ydG4Lfo9nnJO0ZutPz1c7W5ZB_R99EY4oT3oOsLKf4wVwxJ8Bw03antUM89ORm1qoTd-JMS10uw1loHzIiOwNFhdwCtPiExXJsg84UxRwAhx71XoDG0iKiPdqGSVLxVaRjF-DJQ9aGnDkyPwybfCcQdRt6xy4qZqruJ0A5HSVhxKRPjGUlb3gK2bc_cEdWr0KcSjjh4LSmYrtmZ3UIgW3Af0mQSKfHSyQsLRqkWJRrW6lk5foJZc-wQ48NBhq8FSP9Eg87INwW-Tom8irWKQp86tz4VHjnfgWIyYMjv-epxQ7BVd7Jy1s8L3qbcwz3hUlDQ
```


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
$ docker-compose run --rm build
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

- You should see the familiar generated output below:

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

## How to publish on AWS S3

- You can publish to DigitalOcean Spaces or AWS S3, they should work the same. However, auth system
  is only possible with AWS S3 + CloudFront + Lambda@Edge

- Create an AWS 3 bucket, get the right credentials to update the bucket:

```bash
$ export AWS_ACCESS_KEY_ID=<fill_yours>
$ export AWS_SECRET_ACCESS_KEY=<fill_yours>
$ export AWS_BUCKET=<fill_yours>
$ rm -rf web # clean up if needed
$ docker-compose run --rm build # build if needed
$ docker-compose run --rm publish # sync
```

## How to add JWT auth for /dist/

- TODO(hoatle): add docs how to configure the CloudFront + Lambda@Edge.

- The recommend way on how users should authenticate with provided JWT token before `composer install`:

```bash
$ composer config --global bearer.<domain> <token>
```

For example:

```bash
$ composer config --global bearer.d20lvdml4v7ztx.cloudfront.net eyJraWQiOiIxWlowN2FMVF9IOGVnc3JSU1VvMmVkc2dzbUtNVTJCRzBhSmZGZFNiWF9VIiwiYWxnIjoiUFMyNTYifQ.eyJpc3MiOiJodHRwczovL2R4cHIuY29tIiwic3ViIjoidXNlci1pZCIsImF1ZCI6Imh0dHBzOi8vcGFja2FnZXMuZHhwci5jb20iLCJzY29wZSI6Ijg6ZHhwci9keHByX2J1aWxkZXI6KiA5OmR4cHIvZHhwcl9idWlsZGVyOioiLCJpYXQiOjE1ODg2MTMwNzB9.YPrGULY4TUm8Ck6CXU1ydG4Lfo9nnJO0ZutPz1c7W5ZB_R99EY4oT3oOsLKf4wVwxJ8Bw03antUM89ORm1qoTd-JMS10uw1loHzIiOwNFhdwCtPiExXJsg84UxRwAhx71XoDG0iKiPdqGSVLxVaRjF-DJQ9aGnDkyPwybfCcQdRt6xy4qZqruJ0A5HSVhxKRPjGUlb3gK2bc_cEdWr0KcSjjh4LSmYrtmZ3UIgW3Af0mQSKfHSyQsLRqkWJRrW6lk5foJZc-wQ48NBhq8FSP9Eg87INwW-Tom8irWKQp86tz4VHjnfgWIyYMjv-epxQ7BVd7Jy1s8L3qbcwz3hUlDQ
```

- The 2nd possible way on how users could authenticate with provided JWT token when being prompted, they
  must fill in the username with the provided JWT token and password as "bearer", for example:

```bash
Gathering patches for dependencies. This might take a minute.
  - Installing dxpr/dxpr_builder (1.4.4): Downloading (0%)    Authentication required (d20lvdml4v7ztx.cloudfront.net):
      Username: eyJraWQiOiIxWlowN2FMVF9IOGVnc3JSU1VvMmVkc2dzbUtNVTJCRzBhSmZGZFNiWF9VIiwiYWxnIjoiUFMyNTYifQ.eyJpc3MiOiJodHRwczovL2R4cHIuY29tIiwic3ViIjoidXNlci1pZCIsImF1ZCI6Imh0dHBzOi8vcGFja2FnZXMuZHhwci5jb20iLCJzY29wZSI6Ijg6ZHhwci9keHByX2J1aWxkZXI6KiA5OmR4cHIvZHhwcl9idWlsZGVyOioiLCJpYXQiOjE1ODg2MTMwNzB9.YPrGULY4TUm8Ck6CXU1ydG4Lfo9nnJO0ZutPz1c7W5ZB_R99EY4oT3oOsLKf4wVwxJ8Bw03antUM89ORm1qoTd-JMS10uw1loHzIiOwNFhdwCtPiExXJsg84UxRwAhx71XoDG0iKiPdqGSVLxVaRjF-DJQ9aGnDkyPwybfCcQdRt6xy4qZqruJ0A5HSVhxKRPjGUlb3gK2bc_cEdWr0KcSjjh4LSmYrtmZ3UIgW3Af0mQSKfHSyQsLRqkWJRrW6lk5foJZc-wQ48NBhq8FSP9Eg87INwW-Tom8irWKQp86tz4VHjnfgWIyYMjv-epxQ7BVd7Jy1s8L3qbcwz3hUlDQ
      Password: 
Downloading (100%)Do you want to store credentials for d20lvdml4v7ztx.cloudfront.net in /tmp/auth.json ? [Yn] y
```

The 2nd way is not recommended as the token is visible (see: https://github.com/composer/composer/issues/8888)



## How to publish on DigitalOcean Spaces (Deprecated)

- You can publish to DigitalOcean Spaces or AWS S3, they should work the same. However, auth system
  is only possible with AWS S3 + Cloud Front + Lambda@Edge

- Follow:
  + https://www.digitalocean.com/docs/spaces/resources/s3cmd/
  + https://www.digitalocean.com/docs/spaces/resources/s3cmd-usage/

- Then:

```bash
$ export AWS_ACCESS_KEY_ID=<fill_yours>
$ export AWS_SECRET_ACCESS_KEY=<fill_yours>
$ export AWS_BUCKET=<fill_yours>
$ rm -rf web # clean up if needed
$ docker-compose run --rm build # build if needed
$ docker-compose run --rm publish # sync
$ docker-compose run --entrypoint=/bin/sh publish -c "s3cmd setacl s3://$AWS_BUCKET --acl-public --recursive" # set acl if needed
```
