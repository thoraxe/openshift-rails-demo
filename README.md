# openshift-rails-demo
A repository containing various steps and information required for conducting an
OpenShift demo of Ruby/Rails/Jenkins.

## Overview, Background, Requirements
The OpenShift `ruby` cartridge documentation can be found at:
https://github.com/openshift/origin-server/tree/master/cartridges/openshift-origin-cartridge-ruby/README.md

This repository has various tags. Each tag can be used to represent a different
step in the demonstration process. Some user interaction is required, other than
just checking out tags.

The flow of the demonstration is as follows:

1. Create the initial application in OpenShift, and show that it is
reachable/exists.

2. Add this repository as a remote and fetch/merge.

3. Manipulate the files and check out the various tags for each step in the
demo.

We will assume that the OpenShift user's namespace is "myapps" and that the
domain of their PaaS is "paas.example.com".

This demo makes the assumption that you have the rhc tool available and are
comfortable with its use. It also assumes your OpenShift environment can reach
the internet, which is required for fetching the various Ruby gems.

## Initial Application
This repository is designed with the Ruby 1.9 cartridge in mind, and is based
off of the openshift-rails-example repository:

https://github.com/openshift/rails-example

The first step in conducting this demo is to create an initial Ruby 1.9-based
application. For the purposes of these instructions, we will assume that the
application is named "railsdemo" and that the application is created using the
Broker's WebUI.

The resulting application URL would be something like:

http://railsdemo-myapps.paas.example.com

Visit the application to demonstrate that it is accessible already.

Clone the git repo for the application to your local machine.

## The Merge

## First Edit
The first edit is simply to show that we are making changes to a "real"
environment and not just faking it.

### Edit the config.ru File
config.ru is the "rackup" file. Ruby applications in OpenShift are rack-based
applications. What that means is outside of the scope of this demo, but you can
read more about Ruby/Rack here:

https://rack.github.io/

Edit the config.ru file and simply do a search/replace for "OpenShift" with
something else, for example, "MyApp".

Sa

