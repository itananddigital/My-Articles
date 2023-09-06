### Clone frappe_docker and switch directory

```shell
git clone https://github.com/frappe/frappe_docker
cd frappe_docker
```

### Load custom apps through json

`apps.json` needs to be passed in as build arg environment variable.

```shell
export APPS_JSON='[
    {
        "url": "https://github.com/frappe/payments",
        "branch": "develop"
    },
    {
        "url": "https://github.com/frappe/erpnext",
        "branch": "version-14"
    },
    {
        "url": "https://github.com/frappe/hrms",
        "branch": "version-14"
    },
    {
        "url": "https://github.com/itanand/custom_app.git",
        "branch": "master"
    }
]'
export APPS_JSON_BASE64=$(echo ${APPS_JSON} | base64 -w 0)
```


Note:

- `url` needs to be http(s) git url with token/auth in case of private repo.
- add dependencies manually in `apps.json` e.g. add `payments` if you are installing `erpnext`
- use fork repo or branch for ERPNext in case you need to use your fork or test a PR.

### Build Image

```shell
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-14 \
  --build-arg=PYTHON_VERSION=3.10.12 \
  --build-arg=NODE_VERSION=16.20.1 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=docker_username/custom:1.0.0 \
  --file=images/custom/Containerfile .
```

 PR.
- Make sure `APPS_JSON_BASE64` variable has correct base64 encoded JSON string. It is consumed as build arg, base64 encoding ensures it to be friendly with environment variables. Use `jq empty apps.json` to validate `apps.json` file.
- Make sure the `--tag` is valid image name that will be pushed to registry. See section [below](#use-images) for remarks about its use.
- Change `--build-arg` as per version of Python, NodeJS, Frappe Framework repo and branch
- `.git` directories for all apps are removed from the image.

### Push image to use in yaml files

Login to `docker`

```shell
docker login
```

Push image

```shell
docker docker_username/custom:1.0.0
```
