---
layout: page
title: Tutorial
permalink: /tutor/
---

# Oozie

If you are not yet familiar with Oozie, a workflow engine for Hadoop, please,
read about it [here]({{site.baseurl}}{% link oozie.md %}), to get a quick
review of ideas behind it.

# Create a project

To create a project just initialize an empty git repository and add origin to
it. The URL of the origin you can take from datamot UI in the main menu.

```
datamot@localhost:~> cd ~
datamot@localhost:~> mkdir happy-project
datamot@localhost:~> cd happy-project
datamot@localhost:~/happy-project> git init
Initialized empty Git repository in /home/datamot/happy-project/.git/
```

# Add remote repository

Go to datamot UI: [http://localhost:2718](http://localhost:2718). Enter any
username you want. Currently login screen is just a dummy page. Copy remote url
from the main menu panel by just clicking on it. Add remote:

```
datamot@localhost:~/happy-project> git add remote origin datamot@localhost:/home/datamot/.datamot/datamot/babymot
```

# Project structure

Project structure is not dictated. No predefined "special" places for
coordinators, datasets, workflows, scripts etc. You can place any coordinator to
any folder within your project. This means that you can structure your project
anyway you want.

The only thing to remember are "dot" files and "dot" folders. They are not
processed by generating engine.

# Variables substitution and plugins

The real power of datamot is in variables substitution and plugins. In it's
core, datamot is a rendering engine. This means that you can place all the
variables in some "dot" file, namely ".conf" file, and then use them all over
the project. The "scope" of the ".conf" file is a folder in which it placed and
all the folders inside it. You can redefine variables in any folder.

Consider simple project:

<div class="ui segment">
	<div class="ui top attached label">Happy project</div>
	<div class="ui list">
		<div class="item">
			<i class="folder icon"></i>
			<div class="content">
				<div class="header">jobs/</div>
				<div class="list">
					<div class="item">
						<i class="folder icon"></i>
						<div class="content">
							<div class="header">import-happiness/</div>
							<div class="list">
								<div class="item">
									<i class="file icon"></i>
									<div class="content"><a class="header pfile" data-pfile="project-f1">coordinator.xml</a></div>
								</div>
								<div class="item">
									<i class="file icon"></i>
									<div class="content"><a class="header pfile" data-pfile="project-f2">workflow</a></div>
								</div>
								<div class="item">
									<i class="file icon"></i>
									<div class="content"><a class="header pfile" data-pfile="project-f3">.conf</a></div>
								</div>
							</div>
						</div>
					</div>
					<div class="item">
						<i class="folder icon"></i>
						<div class="content">
							<div class="header">calculate-joy/</div>
							<div class="list">
								<div class="item">
									<i class="file icon"></i>
									<div class="content"><a class="header pfile" data-pfile="project-f4">coordinator.xml</a></div>
								</div>
								<div class="item">
									<i class="file icon"></i>
									<div class="content"><a class="header pfile" data-pfile="project-f5">workflow.xml</a></div>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
		<div class="item">
			<i class="folder icon"></i>
			<div class="content">
				<div class="header">datasets/</div>
				<div class="list">
					<div class="item">
						<i class="file icon"></i>
						<div class="content"><a class="header pfile" data-pfile="project-f6">joyful.xml</a></div>
					</div>
					<div class="item">
						<i class="file icon"></i>
						<div class="content"><a class="header pfile" data-pfile="project-f7">painful.xml</a></div>
					</div>
				</div>
			</div>
		</div>
		<div class="item">
			<i class="file icon"></i>
			<div class="content"><a class="header pfile" data-pfile="project-f8">.conf</a></div>
		</div>
	</div>
</div>


<script type="text/javascript">
	$(document).ready(function() {
		$(".project-f").hide()
		$(".pfile").on("click", function() {
			var pfclass = $(this).attr("data-pfile")
			$(".project-f").hide()
			$("." + pfclass).show()
			console.log(pfclass)
		})
	})
</script>

{:.project-f.project-f1}
```xml
<coordinator-app  name="import-happiness"
                  frequency="${coord:days(1)}"
                  start="{{start_date_for.import_happiness}}"
                  end="{{end_date_for.import_happiness}}"
                  timezone="UTC"
                  xmlns="uri:oozie:coordinator:0.2">

  <controls>
    <timeout>{{timeout}}</timeout>
    <execution>FIFO</execution>
  </controls>

  <datasets>
    <include>{{PROJECT_DIR}}/datasets/joyful.xml</include>
  </datasets>

  <output-events>
    <data-out name="happiness" dataset="happiness">
        <instance>${coord:current(0)}</instance>
    </data-out>
  </output-events>

  <action>
    <workflow>
      <app-path>{{PROJECT_DIR}}/jobs/import-happiness/workflow.xml</app-path>
      <configuration>
        <property>
          <name> happiness </name>
          <value> ${coord:dataOut('happiness')} </value>
        </property>
      </configuration>
    </workflow>
  </action>

</coordinator-app>
```

{:.project-f.project-f2}
```xml
<coordinator-app  name="calculate-joy"
                  frequency="${coord:days(1)}"
                  start="{{start_date_for.calculate_joy}}"
                  end="{{end_date_for.calculate_joy}}"
                  timezone="UTC"
                  xmlns="uri:oozie:coordinator:0.2">

  <controls>
    <timeout>{{timeout}}</timeout>
    <execution>FIFO</execution>
  </controls>

  <datasets>
    <include>{{PROJECT_DIR}}/datasets/joyful.xml</include>
  </datasets>

  <input-events>
    <data-in name="happiness" dataset="happiness">
      <instance>${coord:current(0)}</instance>
    </data-in>
  </input-events>

  <output-events>
    <data-out name="joy" dataset="joy" >
      <instance>${coord:current(0)}</instance>
    </data-out>
  </output-events>

  <action>
    <workflow>
      <app-path>{{PROJECT_DIR}}/jobs/calculate-joy/workflow.xml</app-path>
      <configuration>
        <property>
          <name> happiness </name>
          <value> ${coord:dataIn('happiness')} </value>
        </property>
        <property>
          <name> joy </name>
          <value> ${coord:dataOut('joy')} </value>
        </property>
      </configuration>
    </workflow>
  </action>

</coordinator-app>
```

This is a familiar Oozie syntax for a coordinator. But it's too verbose due to
being xml syntax. It is possible to use much shorter and laconic code to
describe coordinators. Refer to [docs]({{site.baseurl}}{% link docs.md %}) to
see how to use plugins.
