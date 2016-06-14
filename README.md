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
1. Get the UUID from the bosh director and edit the manifest file.
'''bosh status'''

'''
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
'''

