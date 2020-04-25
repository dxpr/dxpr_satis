# How to set up github actions

This is used for repository update automation.

## Build

- Make sure to follow the README.md to set up ssh keys so you know how it works locally.

- Now, create secrets on this github repository:

  + Create the `RSA_PRIVATE_KEY_BASE64` secret key with the value from `$ cat .ssh/repository_rsa | base64; echo`.

  + Create the `RSA_PUBLIC_KEY_BASE64` secret key with the value from `$ cat .ssh/repository_rsa.pub | base64; echo`.

  + Create the `GH_PAT` secret key with the value that you created.

After this, try to run github actions to build.

## Publish

- Make sure to follow the README.md so you know how it works locally to publish the built files.

- Now, create secrets on this github repository:
  + Create the `AWS_ACCESS_KEY_ID` secret key with the value that you created.
  + Create the `AWS_SECRET_ACCESS_KEY` secret key with the value that you created.
  + Create the `AWS_BUCKET` secret key with the value that you created.

After this, try to run github actions to publish.
