# sonic-custom-build

sonic-custom-build is shell script for build SONiC image with local changes.


## Prerequisites

- `docker` for build SONiC image.
- `git` for fetch SONiC source code and local changes.
- `make` for build SONiC image.
- `jq` for read specification of local changes.
- `gh` for release built images.


## Basic usage

Basic build target are `build-all`.

```
./sonic-custom-build build-all
```

It build SONiC images for broadcom and vs platform on master and 202305 branch.
It clone repository on current directory if not specified.

Specify branch[es] with `-b` option.  Comma saparated values are accepted.

```
# build only on master branch.
./sonic-custom-build -b master build-all

# build on 202305 and 202211 branch.
./sonic-custom-build -b 202305,202211 build-all
```


Specify platform with '-p' option.  Comma saparated values are accepted.

```
# build for Barefoot platform.
./sonic-custom-build -p barefoot build-all

# build for Mellanox and VS platform.
./sonic-custom-build -p mellanox,vs build-all
```

To upload build images, use target `all`.

```
# build broadcom and vs images and upload release on your repository.
./sonic-custom-build all
```

## Advanced usage

- `-U` option specify user account of SONiC image.
- -`P` option speficy password of user accont of SONiC image.
- `-j` option sprficy parallel build jobs.
- `-w` option speficy working directory.
- `-o` option speficy output directory.

And more options.  Run `./sonic-custom-build --help` for full usage.


## Stages

`./sonic-custom-build` can specify build stage.  `all` means run all stages.
`build-all` means run all stages except for `release`.

- `prepare`     - Check requirement tools.
- `clone  `     - `git clone` from `sonic-net` repository.
- `init`        - Run `make init`.
- `merge_pr`    - Apply pull request.
- `cherry_pick` - Apply individual commit by id.
- `merge`       - Apply local changes.
- `configure`   - Run `make configure`.
- `clean`       - Run `make clean`.
- `build`       - Build SONiC image[es] and `git tag` if successful.
- `release`     - Run `gh release create` on your repository.


## Apply local changes

Write file `config.json` for specification of local changes.
Local changes should be put in remote git repository.
`sonic-custom-build` calls `git fetch` and `git merge` with it.

### Configuration keys

- `repository` - Speficy original base URL (sonic-buildimage).
- `local-respoitory` - Spefify put local changes base URL.
- `https-proxy` - Speficy https proxy if needed.
- `local-changes` - Speficy repository and topic branches for local changes.
  - "NAME": - Speficy repository
    - "TOPIC-BRANCH-NAME" - Specify topic branch for local changes.
- `pull-requests` - Speficy pull requests in original repository.
  - "NAME": - Speficy repository
    - "NUMBER" - Spcify pull requset.

JSON example:

```
{
    "local-repository": "https://github.com/iMasaruOki",
    "remote-name": "mine",
    "pull-requests": {
        "sonic-buildimages": [
	  "17104"
        ]
    },
    "local-changes": {
	"sonic-buildimage": [
	    "keepalived",
	    "copp-vrrp"
	],
	"sonic-buildimage/src/sonic-swss": [
	    "vlan-sub-intf",
	    "vxlan-ttl",
	    "nlmsg_newaddr"
	]
}
```
