# How to set up github actions

This is used for repository update automation.


## Add secrets for env vars

- First, make sure to follow the README.md to set up ssh keys so you know how it works locally.

- Now, create secrets on this github repository.

- Create the `RSA_PRIVATE_KEY_BASE64` secret key with the value from `$ cat .ssh/repository_rsa | base64; echo`

- Create the `RSA_PUBLIC_KEY_BASE64` secret key with the value from `$ cat .ssh/repository_rsa.pub | base64; echo`

- Create the `GH_PAT` secret key with the value that you created
