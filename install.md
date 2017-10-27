---
layout: page
title: Installation How To
permalink: /install/
---

# Installtion How To

Normally you need to install datamot to some server which is available to push
via ssh for you and your teammates. Your hadoop cluster and it's services, like
Oozie, HDFS need to be available from this server. And it would be just nice to
create a dedicated user on that server to run datamot.

After creating a user (or deciding to use an existing one) unpack datamot
to whatever place you want, and add it's `bin` folder to paths. So, if, for
example, you unpack it to `/home/datamot/datamot`, then you need to fix $PATH
environment variable like this:

`$PATH=/home/datamot/datamot/bin/:$PATH`

Now `datamot --version` should work from anywhere. If your are going to use UI
(we adwise you to use it) just run `datamot --start`. Currently we are working
on installing datamot as a service. For now you can use screen to run detached
datamot like this:

`screen -dmS datamot datamot --start`

You can change default ip and port on which datamot will listen. Take a look at
`conf/datamot.conf`.

`remoteUser` is a ssh user under which you are going to push to git,
`remoteHost` is a host to push,

these two variables are needed to show push URL in the user interface.
