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

