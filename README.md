Play framework 2 Scala application on OpenShift Express
============================

This git repository will help you get up and running quickly with a Play framework 2 (with Scala) application
on OpenShift Express taking advantage of the do-it-yourself cartridge.


Running on OpenShift
--------------------

Register at http://openshift.redhat.com/, and then create a diy (do-it-yourself) application:

    rhc app create -a play2scala -t diy-0.1

Add this upstream play-example repo:

    cd play2scala
    git remote add upstream -m master git://github.com/opensas/play2-scala-openshift-quickstart.git
    git pull -s recursive -X theirs upstream master

Now you should update your appName setting in project/Build.scala to match your application name.

And use the stage task to prepare your deployment

    play clean compile stage

Then add your changes to git's index, commit and push the repo upstream:

    git add .
    git commit -m "a nice message"
    git push origin

That's it, you can now see your application running at:

    http://play2scala-yournamespace.rhcloud.com

The first time you do it, it will take quite a few minutes to complete, because git has to upload play's dependencies, but after that git is smart enough to just upload the differences.

Working with a mysql database
----------------------------

Just issue:

    rhc app cartridge add -a play2scala -c mysql-5.1

Don't forget to write down the credentials.

Then uncomment the following lines from your conf/application.conf, like this:

    # openshift mysql database
    %openshift.db.url=jdbc:mysql://${OPENSHIFT_DB_HOST}:${OPENSHIFT_DB_PORT}/${OPENSHIFT_APP_NAME}
    %openshift.db.driver=com.mysql.jdbc.Driver
    %openshift.db.user=admin
    %openshift.db.pass=<write your password here>

You can manage your new MySQL database by embedding phpmyadmin-3.4.

    rhc app cartridge add -a play2scala -c phpmyadmin-3.4

It's also a good idea to create a different user with limited privileges on the database.

Updating your application
-------------------------

To deploy your changes to openshift just run the stage task, add your changes to the index, commit and push:

```bash
    play clean compile stage
    git add . -A
    git commit -m "a nice message"
    git push origin
```

If you want to do a quick test, you can skip the "clean compile" stuff and just run "play stage"

All right, I know you are lazy, just like me. Si I added a little script to help you with that, just run

```bash
    rhc_deploy "a nice message"
```

You may leave the message empty and it will add something like "deployed on Thu Mar 29 04:07:30 ART 2012", you can also pass a "-q" parameter to skip the "clean compile" option.

Deploying an existing application to openshift
-------------------------

You can add openshift support to an already existing play application. Let's take the forms application.

```bash
    cd PLAY_INSTALL_FOLDER/samples/scala/forms

    git init
    rhc app create -a forms -t diy-0.1 --nogit
```

We add the "--nogit" parameter to tell openshift to create the remote repo but don't pull it locally. You'll see something like this:

```bash
    Confirming application 'forms' is available:  Success!

    zentasks published:  http://forms-opensas.rhcloud.com/
    git url:  ssh://uuid@forms-yournamespace.rhcloud.com/~/git/forms.git/
    Disclaimer: This is an experimental cartridge that provides a way to try unsupported languages, frameworks, and middleware on Openshift.
```
So we will manually add it as a remote repo

    git remote add origin ssh://uuid@forms-yournamespace.rhcloud.com/~/git/forms.git/

And the rest is just the same

    git remote add upstream -m master git://github.com/opensas/play2-scala-openshift-quickstart.git
    git pull -s recursive -X theirs upstream master
    play clean compile stage

Then add your changes to git's index, commit and push the repo upstream:

    git add .
    git commit -m "deploying forms application"
    git push origin


That's it, you can now see zentasks demo application running at:

    http://zentasks-yournamespace.rhcloud.com

Trouble shooting
----------------------------

To find out what's going on in openshift, issue

    rhc app tail -a play2scala

If you feel like investigating further, you can

    rhc app show -a play2scala

    Application Info
    ================
    play
        Framework: raw-0.1
        Creation: 2012-03-18T12:39:18-04:00
        UUID: youruuid
        Git URL: ssh://youruuid@play-yournamespace.rhcloud.com/~/git/raw.git/
        Public URL: http://play-yournamespace.rhcloud.com

Then you can connect using ssh like this:

    ssh youruuid@play-yournamespace.rhcloud.com


Configuration
-------------

When running on openshift, the configuration defined with conf/application.conf will be overriden by conf/openshift.conf, that way you can customize the way your play app will be executed while running on openshift.

You can also specify additional parameters to pass to play's executable with the **openshift.play.params** key, like this:

    # play framework command configuration
    # ~~~~~
    #openshift.play.params=-Xmx512M


Having a look under the hood
----------------------------

This projects takes advantage of openshift's do-it-yourself cartridge to run play framework 2 application natively.

Everytime you push changes to openshift, the following actions will take place:

* Openshift will run the **.openshift/action_hooks/stop** script to stop the application, in case it's running.

* Then it wil execute **.openshift/action_hooks/start** to start your application. You can specify additional parameters with openshift.play.params.

```bash
    ${OPENSHIFT_REPO_DIR}target/start $PLAY_PARAMS 
        -Dhttp.port=${OPENSHIFT_INTERNAL_PORT}
        -Dhttp.address=${OPENSHIFT_INTERNAL_IP}
        -Dconfig.resource=openshift.conf
```

Play will then run your app in production mode. The server will listen to ${OPENSHIFT_INTERNAL_PORT} at ${OPENSHIFT_INTERNAL_IP}.

* **.openshift/action_hooks/stop** just tries to kill the RUNNING_PID process, and then checks that no "java" process is running. If it's there, it tries five times to kill it nicely, and then if tries another five times to kill it with -SIGKILL.

Acknowledgments
----------------------------

I couldn't have developed this quickstart without the help of [marekjelen](https://github.com/marekjelen) who answered [my questions on stackoverflow](http://stackoverflow.com/questions/9446275/best-approach-to-integrate-netty-with-openshift) and who also shared his [JRuby quickstart repo](https://github.com/marekjelen/openshift-jruby#readme). (I know, open source rocks!)

It was also of great help Grant Shipley's [article on building a quickstart for openshift](https://www.redhat.com/openshift/community/blogs/how-to-create-an-openshift-github-quick-start-project).

Play framework native support for openshift was a long awaited and pretty popular feature (you are still on time to vote for it [here](https://www.redhat.com/openshift/community/content/native-support-for-play-framework-application)). So it's a great thing that Red Hat engineers came out with this simple and powerful solution, that basically let's you implement any server able to run on a linux box. Kudos to them!!!

Licence
----------------------------
This project is distributed under [Apache 2 licence](http://www.apache.org/licenses/LICENSE-2.0.html). 