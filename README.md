# Central Package Repository

### Current application state
This application currently is modified to only create hashes for NuGet and store them in a file.


### How to run

__Requirements:__

- You will need access to a redis server. If you're not running one, you can spin up one using docker.
  ```bash
    docker run --name redis-server -p 6379:6379  -d redis
  ```

- run the following command to start the worker queue. _You may pause the worker queue at any time by simply exiting out of it (ctrl+c). It is configured to gracefully pause by completing the in flight items and shutting down._ 
  You can start it again at any point to continue processing items from the queue. To completely reset the queue, you can simply kill the redis docker container and re-create it.
  ```bash
    node index.js
  ```

- in a separate terminal run the NuGet test script. 
**NOTE:** _As is, this script will only grab the first page of the catalog, however that is stil a huge amount of data. You should further modify it to reduce the number of packages it iterates over. Or kill the process shortly after you start it._
  ```bash
    node test.js
  ```

Overview:

- `index.js` starts the worker and adds listeners for jobs. completed jobs data will be appended to `results.json`

- `test.js` runs a script that will iterate over the NuGet catalog, identifies the packages and their versions, and queues each version to be downloaded.
 
- `processor.js` is called from workers specified in `worker.js` and downloads the nupkg files in the `nupkg` directory which is currently not getting cleaned, unzips it and for every file generates cryptographic hashes and returns the result.

- see `config.js` to update concurrency, connection, ...

- `schema.sql` has the current schema definition that is used by other applications. This schema is not currently used in this project and may need modifications.

---


Concepts:

  - __Registry/Package Manager__: 
    - __Package__: Packages are registered under a registry
        - latest metadata (name, version, ...)
        - __Versions__: if registry doesn't support versioning? libraries.io attempts to use git tags - we can either use that or attempt versioning it ourselves.
          - __Dependencies__: Dependencies per version; also type of dependency (dev, prod, ...)
      - __Repository__: Repositories are tied to packages
        - __Tags__: Possible correlation to versions/especially if registry doesn't support versions
        - __Contributors & authors__: ???
        - __License__: ???


Requirements:

- Almost everything should be tracked at the version step to ensure hashes, dependencies, licenses, ... are accurate.

- Every integration should be optimized for performance and accuracy

---


__Data loading:__

For every registry, we will need to perform at least one full load of their data. Depending on the registry, we should store the latest commit/timestamp/... that will allow us to pull data incrementally after.

__Useful resources:__

Nuget API implementation overview: (_This has not been implemented_)

 - https://learn.microsoft.com/en-us/nuget/api/catalog-resource#overview

NPM stream:

 - https://github.com/npm/registry-follower-tutorial
 
 - https://github.com/jcrugzz/changes-stream
 
 - https://github.com/nice-registry/nice-package