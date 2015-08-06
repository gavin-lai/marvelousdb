## Appfog and Orchestrate Marvel Sample Application for [CenturyLink Cloud](//www.ctl.io)

All Comics and Characters in this sample application are © 2015 MARVEL

### Prerequisites

* A [CenturyLink Cloud](//www.ctl.io) Account
* [Access](//www.ctl.io/knowledge-base/appfog/manage-appfog-membership/) to [AppFog](//www.ctl.io/appfog) Services
* [Atom](//atom.io) Text Editor
* [Go](//golang.org/dl/)
* [CloudFoundry CLI](//github.com/cloudfoundry/cli/releases)
* [Github Windows](//windows.github.com/) or [Github Mac](//mac.github.com/)

### Deploy the App

1. Clone the [Marvel Sample Application.](//github.com/chrislittle/marvelousdb)  *Tip: data will be cloned into your current working directory/marvelousdb*

    ```
    git clone https://github.com/chrislittle/marvelousdb.git
    ```

    ```
    Cloning into 'marvelousdb'...
    remote: Counting objects: 1223, done.
    remote: Compressing objects: 100% (1212/1212), done.
    remote: Total 1223 (delta 3), reused 1220 (delta 3), pack-reused 0
    Receiving objects: 100% (1223/1223), 16.28 MiB | 3.42 MiB/s, done.
    Resolving deltas: 100% (3/3), done.
    Checking connectivity... done.
    Checking out files: 100% (1209/1209), done.
    ```

2. [Login to Appfog](//www.ctl.io/knowledge-base/appfog/login-using-cf-cli/) and select an AppFog space.

    ```
    cf login -a <API ENDPOINT> -o <ALIAS> -u <CONTROL USERNAME>
    ```

3. Change your local working directory to the newly Cloned *Marvelousdb* directory.

4. Create a new AppFog space to place the application.  *Tip: For educational reasons you may use your name for this newly created space making it easy to clean up and build apps in your own working area.*

    ```
    cf create-space <space name>
    ```

5. Target this newly created space.

    ```
    cf target -s <space name>
    ```

6. Create the [Orchestrate](//orchestrate.io) DBaaS service in AppFog.  *Tip: you can view a list of services available in the AppFog marketplace by using the 'cf marketplace' command.*

    ```
    cf create-service orchestrate free marveldb
    ```

    Note: the 'marveldb' service name matches the manifest.yml in this sample application.  Use of a different name for the DBaaS service requires the user edit the manifest.yml file appropriately prior to pushing the application in later steps.

    ```
    manifest.yml
    ---
    applications:
    - name: marveluniverse-${random-word}
      memory: 256MB
      instances: 1
      services:
        - marveldb
    ```

7. Push your Marvel Application to AppFog.  The manifest.yml simplifies the process by defining the app name (with a random word), memory, instance counts and the services (in this case Orchestrate DBaaS) to which the application will bind.

    ```
    cf Push
    ```

    ```
    requested state: started
    instances: 1/1
    usage: 256M x 1 instances
    urls: marveluniverse-<Random-Word>.useast.appfog.ctl.io
    last uploaded: Mon Jul 27 16:20:50 UTC 2015
    stack: cflinuxfs2
    buildpack: Node.js

            state     since                    cpu    memory          disk          details
      #0   running   2015-07-27 12:21:30 PM   0.0%   46.3M of 256M   57.2M of 1G
    ```

8. You can validate your application is functional by using the 'cf apps' command to gather the URL.  Alternatively, the URL is available for the application the Control Portal. You'll notice that while the application is functional there is no data loaded into the database layer yet.

    ```
    name                           requested state   instances   memory   disk   urls
    marveluniverse-<Random-Word>   started           1/1         256M     1G     marveluniverse-<Random-Word>.useast.appfog.ctl.io
    ```

### Bulk Upload Data to Orchestrate DBaaS

1. Clone the [Orchestrate Bulk Import Tool.](//github.com/chrislittle/orchestrate-bulk-import.git) *Tip: data will be cloned into your current working directory/orchestrate-bulk-import*

    ```
    git clone https://github.com/chrislittle/orchestrate-bulk-import.git
    ```

    ```
    Cloning into 'orchestrate-bulk-import'...
    remote: Counting objects: 6, done.
    remote: Compressing objects: 100% (5/5), done.
    remote: Total 6 (delta 0), reused 3 (delta 0), pack-reused 0
    Unpacking objects: 100% (6/6), done.
    Checking connectivity... done.
    ```

2. Change your local working directory to the newly Cloned *orchestrate-bulk-import* directory.

3. Run 'cf apps' to capture the name of your running application.

    ```
    cf apps
    ```

    ```
    name                           requested state   instances   memory   disk   urls
    marveluniverse-<Random-Word>   started           1/1         256M     1G     marveluniverse-<Random-Word>.useast.appfog.ctl.io
    ```

4. run 'cf env app_name' to gather the API_KEY and API_URL for use with the Orchestrate Bulk Import tool.

    ```
    cf env marveluniverse-<Random-Word>
    ```

    ```
    System-Provided:
    {
      "VCAP_SERVICES": {
        "orchestrate": [
    {
      "credentials": {
        "ORCHESTRATE_API_HOST": "api.ctl-va1-a.orchestrate.io",
        "ORCHESTRATE_API_KEY": "API_KEY_STRING",
        "ORCHESTRATE_API_URL": "https://api.ctl-va1-a.orchestrate.io/"
        },
        "label": "orchestrate",
        "name": "marveldb",
        "plan": "free",
        "tags": []
          }
          ]
        }
      }
    ```

5. Use [Atom](//atom.io) to open 'orcbulkimport.go' located in the orchestrate-bulk-import working directory. Edit the **apiKey** and **host** variables with the data collected from the 'cf env' command.  

      ```
      var (
	      apiKey                = flag.String("key", "API_KEY_STRING", "the api key")
	      workerCount           = flag.Int("workers", 8, "the number of worker procs")
	      host                  = flag.String("host", "api.ctl-va1-a.orchestrate.io", "the Orchestrate API host to use")
	      reqs                  = make(chan Request, 100)
	      dialTimeout           = 3 * time.Second
	      responseHeaderTimeout = 60 * time.Second
	      wg                    sync.WaitGroup
	      client                *http.Client
        )
    ```

6. Included in the orchestrate-bulk-import repository is a zip sample data set.  Unzip **marvelous-sorted.zip** and Execute 'go run orcbulkimport.go marvelous-sorted.json' from the working directory to bulk load the sample Marvel data to Orchestrate.  

    ```
    go run orcbulkimport.go marvelous-sorted.json
    ```

    ```
    2015/07/27 13:11:46 Importing marvelous-sorted.json
    2015/07/27 13:11:56 Progress imported 1000 items from marvelous-sorted.json
    2015/07/27 13:12:00 Progress imported 2000 items from marvelous-sorted.json
    2015/07/27 13:12:05 Progress imported 3000 items from marvelous-sorted.json
    2015/07/27 13:12:09 Progress imported 4000 items from marvelous-sorted.json
    2015/07/27 13:12:14 Progress imported 5000 items from marvelous-sorted.json
    .......
    2015/07/27 13:18:49 Progress imported 125000 items from marvelous-sorted.json
    2015/07/27 13:18:49 Progress imported 126000 items from marvelous-sorted.json
    2015/07/27 13:18:50 Done importing 128029 items from marvelous-sorted.json (with 0 errors)
    ```

7.  Open your AppFog Marvel Application and Explore the Marvel universe.

### Scaling AppFog applications

1. Scale your Marvel Application Memory from 256MB to 512MB.

    ```
    cf scale marveluniverse-<Random-Word> -m 512MB
    ```

2. Scale your Marvel Application Instance count from 1 to 4.

    ```
    cf scale marveluniverse-<Random-Word> -i 4
    ```

### Clean-Up

1. Run 'cf delete app_name' to delete your test application

    ```
    cf delete marveluniverse-<Random-Word>
    ```

    ```
    Really delete the app marveluniverse-<Random-Word>? yes
    Deleting app marveluniverse-<Random-Word> in org <ALIAS> / space <Space Name> as <Username>...
    OK
    ```

2. Run 'cf delete-service marveldb' to remove the Orchestrate DBaaS instance.

    ```
    cf delete-service marveldb
    ```

    ```
    Really delete the service marveldb?> yes
    Deleting service marveldb in org <ALIAS> / space <Space Name> as <Username>...
    OK
    ```

3. Run 'cf delete-space space-name' to remove the AppFog space

    ```
    cf delete-space <space name>
    ```

    ```
    Really delete the space <space name>? yes
    Deleting space <space name> in org <Alias> as <Username>...
    OK
    ```
