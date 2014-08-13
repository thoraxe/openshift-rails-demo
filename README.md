# openshift-rails-demo
A repository containing various steps and information required for conducting an
OpenShift demo of Ruby/Rails/Jenkins.

## Background, Overview, Requirements
The OpenShift `ruby` cartridge documentation can be found at:
https://github.com/openshift/origin-server/tree/master/cartridges/openshift-origin-cartridge-ruby/README.md

This repository has various tags. Each tag can be used to represent a different
step in the demonstration process. Some user interaction is required, other than
just checking out tags. It is strongly advised that you do not refer to this
file while leveraging this demo, as changing the tags may inadvertently change
the state of this file and place you in a strange documentation state.

### Overview
The flow of the demonstration is as follows:

1. Create the initial application in OpenShift, and show that it is
reachable/exists.

2. Add this repository as a remote and fetch/merge.

3. Manipulate the files and check out the various tags for each step in the
demo.

We will assume that the OpenShift user's namespace is "myapps" and that the
domain of their PaaS is "paas.example.com".

### Requirements

This demo makes the assumption that you have the rhc tool available and are
comfortable with its use. It also assumes your OpenShift environment can reach
the internet, which is required for fetching the various Ruby gems.

As installing the rhc tool will install Ruby and rubygems, the only other thing
you will need is the bundler gem.

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
Once you have the skeleton Ruby application repo cloned to your local machine,
it's time to merge this repository in so that you'll be able to use its tags.
Understanding what we're doing is beyond the scope of this demo, as it uses some
interesting features of git. 

Change your directory into the repo for your application and do the following:
```
git remote add demorepo https://github.com/thoraxe/openshift-rails-demo.git
git fetch demorepo
```

You will get an error about conflicts. (resolution instructions)

## First Tag
We need to switch to the first tag so that we can start building up from our
"base".

### Merging
Execute the following command:

```
git merge -Xtheirs v1-base-app
```

This will switch you to the initial vanilla OpenShift Ruby cartridge repository
state. You don't have to edit the merge message at all. After the merge
completes, we can proceed with our first edit.

### Edit the config.ru File
The first edit is simply to show that we are making changes to a "real"
environment and not just faking it.

config.ru is the "rackup" file. Ruby applications in OpenShift are rack-based
applications. What that means is outside of the scope of this demo, but you can
read more about Ruby/Rack here:

https://rack.github.io/

Edit the config.ru file and simply do a search/replace for "OpenShift" with
something else, for example, "MyApp".

### Add, Commit, Push
Now you can add your changes, commit them, and push them up.

```
git add .
git commit -m "changed to MyApp"
git push origin master
```

Once the push is complete, refresh your application and show that, indeed,
OpenShift has changed to MyApp.

## Add MySQL
The next step is to add the MySQL cartridge. We will demonstrate using the RHC
command for this, instead of doing it through the UI.

```
rhc cartridge add mysql-5.1 --app railsdemo
```

Depending on your environment you may need to specify a different OpenShift
config file, or other options.

## Rails
The next step is going to be to "switch" to using Ruby/Rails. We can do this by
checking out a different tag and then pushing up the new repo.

### The git Part
Execute the following (remembering to simply accept the merge commit message):

```
git merge -Xtheirs v2-base-rails
bundle update
git add .
git commit -m "bundled gems and switched to base rails application"
```

### Pushing
Now you'll need to push to origin/master.

```
git push origin master
```

As we are now leveraging more Ruby gems and have a Gemfile.lock, you will see
that OpenShift will begin to build the gems inside your user's gear. When this
is complete, you should be able to refresh the application and see the base
Rails welcome message.

## Flesh out the application
Now that we have our base Rails application going, we can grab another tag that
has swapped out the homepage and done a few other things.

### Merge
We'll again use the merge feature of git to grab version 3:

```
git merge -Xtheirs v3-bank-application
bundle update
git add .
git commit -m "updated gems and now using the base bank application"
```

### Push
Push the latest code changes. You will again see some gem building:

```
git push origin master
```

When this push is complete, if you refresh the application you will see "Welcome
to the bank!" as the only text on the page.

## Rspec, Testing and Jenkins
RSpec is testing tool for the Ruby programming language. Born under the banner
of Behaviour-Driven Development, it is designed to make Test-Driven Development
a productive and enjoyable experience.

You can learn more at:
http://rspec.info/

### Running the Test
We have one simple test that verifies the word "Welcome" appears on the home
page. You can run it and show your audience that it works:

```
bundle exec rspec spec/features/home_spec.rb
```

You should see that there were zero failures. Now we can enable Jenkins,
configure it to run our tests, and purposefully fail a test to demonstrate how
this works.

### Enabling Jenkins
From the Broker's web console, enable Jenkins for your application. If you
didn't already have a Jenkins server in your account, you'll need to do those
set up steps as part of this process.

### Configuring Jenkins
Once your Jenkins is enabled, log into the Jenkins administrative interface and
edit/configure the build that was populated by OpenShift. It will be called
"railsdemo-build".

In the "Build" section of the configure page, inside the "Execute shell" box,
find the following comment and add our two lines below it:

```
# Run tests here
rake db:test:prepare RAILS_ENV=test
bundle exec rake ci:setup:rspec spec RAILS_ENV=test
```

Scroll down further on the page and click the button that says "Add post-build
action" and then select "Publish JUnit test result report". In the box labeled
"Test report XMLs" enter the following:

```
test/reports/*.xml,spec/reports/*.xml
```

Click "Save" when you are finished.

### Purposefully breaking the application
Now that Jenkins is enabled, let's make a quick change to our application to
purposefully break it. We will edit the page that we are testing for "Welcome"
and make sure it does not appear.

Edit the file:
app/views/welcome/index.html.haml

Change the word "Welcome" however you like, as long as "Welcome" no longer
appears anywhere in this file. An easy change is simply making a typo --
"Welcram".

### Add your change and push
Add the changes you just made and push them up to OpenShift:

```
git add .
git commit -m "purposefully breaking the application"
git push origin master
```

If you've done everything right, OpenShift will indicate that it is executing a
Jenkins build. At this point you can return to the Jenkins web interface to
watch your build progress. You will want to look at the console output of the
Jenkins build.

What you should see is that Jenkins will build and install all of the Ruby gems
inside the build area, perform the tests, and hit a failure. Since the build
fails, Jenkins will not deploy this new code to the "real" application running
on OpenShift.

You can verify this by refreshing the application. It should not show "Welcram".

## Conclusion
This concludes the demo of Ruby, Rails and Jenkins on OpenShift.
