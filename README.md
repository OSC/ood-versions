# ood-versions

Simple app to compare versions of OOD apps installed at different OOD sites

Assumptions: 

1. Each sys app installed under /var/www/ood/apps/sys/ has their .git directory included, in order to get information such as the git remote and version deployed

## New Install

1. Clone the app in your dev sandbox and build it

    ```sh
    scl enable git19 -- git clone https://github.com/OSC/ood-versions.git versions
    cd versions
    scl enable rh-ruby22 -- bin/bundle install --path vendor/bundle
    ```

2. Copy the `bin/portal-info.php` to /public of each OOD site

3. Add initializer to specify each portal location and associated build directory (if relevant)

    ```ruby
    # config/initializers/ood-portals.rb

    # OSC
    Portal.portals << Portal.new(site: "OSC", name: "production", url: "https://ondemand.osc.edu", build: "/users/PZS0645/wiag/ood_portals/ondemand/sys")
    Portal.portals << Portal.new(site: "OSC", name: "staging", url: "https://ondemand-test.hpc.osc.edu", build: "/users/PZS0645/wiag/ood_portals/ondemand/sys")
    Portal.portals << Portal.new(site: "OSC", name: "testing", url: "https://ondemand-dev.hpc.osc.edu") # no separate build directory

    # AweSim
    Portal.portals << Portal.new(site: "AweSim", name: "production", url: "https://apps.awesim.org", build: "/users/PZS0645/wiag/ood_portals/awesim/sys")
    ```

4. Access via /pun/dev/versions

## Usage

Go to /pun/dev/versions/install to verify the installation and configuration.

With a properly configured app, you should be able to access different table views comparing of all of the deployed versions of the apps across the different portals.

## Development/Design

`bin/portal-info.php` does the bulk of the work. It is deployed to the public location of an OOD deployment and accessible via /public/portal-info.php. With no query arguments, it will return a JSON array of apps installed at `/var/www/ood/apps/sys`, with each app being a hash of key value pairs:

```json
[
  {
    "name":"bc_osc_abaqus_cae",
    "git_version":"v1.5.0",
    "git_remote_origin_url":"git@github.com:OSC/bc_osc_abaqus_cae.git",
    "git_sha":"ed01092",
    "path":"/var/www/ood/apps/sys/bc_osc_abaqus_cae"
  },
  {
    "name":"bc_osc_oakley_desktop",
    "git_version":"v0.0.1",
    "git_remote_origin_url":"git@github.com:OSC/bc_osc_oakley_desktop.git",
    "git_sha":"74f534e",
    "path":"/var/www/ood/apps/sys/bc_osc_oakley_desktop"
  }
]
```

If a query param `path=` is provided, the full path will be the directory that is used.

**We might consider restricting valid paths to directories under wiag home directory or something similar. Also, this should be sanitized before being used. Otherwise, just hardcode the build path when deploying the portal-info.php.**

Here is example code in ruby that has been used in the past to get this information:

```ruby
  # Get the output of `git describe`
  #
  # @return [String] tag or branch or sha
  def git_version
    `GIT_DIR=#{path}/.git git describe --always --tags`.strip
  end

  # Get the current commit sha
  #
  # @return [String] sha of the HEAD
  def git_sha
    `GIT_DIR=#{path}/.git git rev-parse --short HEAD`.strip
  end

  # Get the url of the remote origin
  #
  # @return [String] url (either ssh or https)
  def git_remote_origin_url
    #FIXME: copied from Product@get_git_remote
    `cd #{path} 2> /dev/null && HOME="" git config --get remote.origin.url 2> /dev/null`.strip
  end
  ```
  
The Passenger ruby app itself will curl or use a simple library to query this PHP file for this information, parse it, and format the results to the user. Bonus if we do this without using external gems - then we can omit the build step.

An automated test will be added to confirm that apps deployed to production in AweSim and OnDemand are identical in version.

Problems with this design:

1. No easy way to do a diff between the files in two deployed directories; for example, if configuration files have changed such as a `.env.local` or a custom initializer.
    * possible solution to this: we could have a list of common app configuration files we know will change, adding to portals-info.php (or even passing these as query params, relative to the app root) and if the files exist (or don't exist) we can have that specified in the JSON; if they do exist, an MD5 hash of the file contents to suggest whether or not the contents are the same

2. Publicly available means we probably shouldn't be using query params (that can be dangerous).
