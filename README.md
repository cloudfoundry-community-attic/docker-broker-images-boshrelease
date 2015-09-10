# BOSH Release for docker-broker-images

This boshrelease can be used to seed a bunch of docker [images](https://github.com/cloudfoundry-community/docker-broker-images-boshrelease/blob/master/images.yml) into a [configurable list of docker nodes](https://github.com/cloudfoundry-community/docker-broker-images-boshrelease/blob/master/jobs/docker_load_images/spec#L10-L16).

## Usage

This boshrelease has been tested to work with [docker-boshrelease](https://github.com/cf-platform-eng/docker-boshrelease).
To get started quickly it is recommended to use [docker-services-boshworkspace](https://github.com/cloudfoundry-community/docker-services-boshworkspace) which comes with some preconfigured deployments.

If you want to include this release in an existing deployment you can add [docker-offline.yml](https://github.com/cloudfoundry-community/docker-broker-images-boshrelease/blob/master/templates/docker-offline.yml] to your spiff templates.

When using the above template the images can be loaded into docker by running:
`bosh run errand fetch-containers`

### Development

As a developer of this release, create new releases and upload them:

```
bosh create release --force && bosh -n upload release
```

### Final releases

To share final releases:

```
bosh create release --final
```

By default the version number will be bumped to the next major number. You can specify alternate versions:


```
bosh create release --final --version 2.1
```

After the first release you need to contact [Dmitriy Kalinin](mailto://dkalinin@pivotal.io) to request your project is added to https://bosh.io/releases (as mentioned in README above).
