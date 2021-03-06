# Deploy nginx with bosh lite
This sample demonstrates how to deploy a lightweight [nginx](https://nginx.org/) server on [bosh lite](https://github.com/cloudfoundry/bosh-lite). Detailed instruction about setup bosh lite can be found on the [project page](https://github.com/cloudfoundry/bosh-lite).

I expect there is a running bosh instance and the target and login process are concluded.

## Steps to create the release

* create a new release with `bosh init release bosh-nginx-release`. The bosh director creates a directory with some files and the following scaffolding. ([docu](http://bosh.io/docs/create-release.html))
```
├── blobs
├── config
│   └── blobs.yml
├── jobs
├── packages
└── src
```
* In my example I'm using a script to put the source files into the src subfolder. See the [`update`](https://github.com/phartz/bosh-nginx-sample/blob/master/update) file.

* A simple nginx server needs only one job to work. To create a new job use `bosh generate job nginx`. ([docu](http://bosh.io/docs/jobs.html))
```
├── jobs
    └── nginx
        └── monit
        ├── spec      
        └── templates
```
* In the templates folder create the file [`ctl.erb`](https://github.com/phartz/bosh-nginx-sample/blob/master/jobs/nginx/templates/ctl.erb). It contains the control commands for the bosh agent to control the services. A stumpling point is how to get the pid from nginx when the server starts. The pid is used to stop the server. Bosh expects a file containg the pid to stop \(`kill -9`\) the process. `nginx -g "pid $PID_FILE;"` ([docu](https://nginx.org/en/docs/switches.html))

* The templates that are transformed at runtime are also located in the templates folder. The [`spec`](https://github.com/phartz/bosh-nginx-sample/blob/master/jobs/nginx/spec) file contains the specification for the jobs.

* After creating the jobs, the packages are needed. Use `bosh generate package nginx` to generate it. ([docu](http://bosh.io/docs/packages.html))
  * The file ['packaging'](https://github.com/phartz/bosh-nginx-sample/blob/master/packages/nginx/packaging) contains the steps to compile and install the package.
  * Use the ['spec'](https://github.com/phartz/bosh-nginx-sample/blob/master/packages/nginx/spec) file to specify the dependencies for the packages.
  
* Now it's time to create the 'manifest.yml'. Against the rules of PaaS, in this version of bosh, the manifest files depends in some points on the underliying infrastructure. F.e. the network and ressource pool, depends on the platform.  [`manifest.yml` for bosh lite](https://github.com/phartz/bosh-nginx-sample/blob/master/examples/manifest_bosh_lite.yml)

## Now deploy the example
### Get the UUID from the bosh director and edit the manifest file.
```
$ bosh status
```

```bash
Config
        /Users/phartz/.bosh_config

Director
  Name       Bosh Lite Director
  URL        https://192.168.50.4:25555
  Version    1.3232.2.0 (00000000)
  User       admin
  UUID       93c01d2d-a2a5-49d8-8d02-389fd223a9cf
  CPI        warden_cpi
  dns        disabled
  compiled_package_cache enabled (provider: local)
  snapshots  disabled

Deployment
```
### Set manifest file 
```
$ bosh deployment ./examples/manifest_bosh_lite.yml
```

```
Deployment set to '/Users/phartz/bosh/workspace/samples/bosh-nginx-sample/examples/manifest_bosh_lite.yml'
```

### Create release
````
$ bosh create release --force
```

```
...
Release name: dev
Release version: 0+dev.1
Release manifest: /Users/phartz/bosh/workspace/samples/bosh-nginx-sample/dev_releases/dev/dev-0+dev.1.yml
```

### Upload release
And now upload it.
````
$ bosh upload release
```

### Deploy release
````
$ bosh deploy
```
With bosh lite it is possible an error occours while deploying a release [`...permission denied @ dir_s_mkdir...`](https://www.evoila.de/2015/04/27/howto-fix-bosh-lite-deployment-with-error-error-100-permission-denied-dir_s_mkdir-vagranttmp/?lang=en).
There is a misconfiguration in the bosh VM. Execute this in the bosh lite vm itself `sudo chown -cR vcap:vcap /vagrant/`.

### Done
That's it! 
With `curl 10.244.0.2` you should be able to reach the nginx server.

### Remarks
Possibly you have to run `add-route.bat` in the bosh-lite binary folder.


