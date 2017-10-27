---
layout: page
permalink: /docs/
---

# Variables

You can define a set of variables in the .conf file in the root folder, and use
them in any place in any file of your project: coordinators, workflows, scripts.
A variable is referred by its name placed inside double curly braces:

```xml
coordinator.xml
...
  <controls>
    <timeout>{{site.O}}timeout{{site.C}}</timeout>
    <execution>FIFO</execution>
  </controls>

  <datasets>
    <include>{{site.O}}PROJECT_DIR{{site.C}}/datasets/raw.xml</include>
  </datasets>
...
```

Configuration variables, like hostnames, schedule dates, ML model parameters
etc. are kept in a `.conf` file in the project root. You can redefine or even
define new variables in any folder inside the project. Just put another `.conf`
file there.

# Profiles

Profile is a set of variables with their values. Profiles are used to deploy the
same project with different values of the same variables. Since you can move
merely anything to a variable (paths, dates, thresholds) profiles are quite
flexible tool to variate your project. For example you may want to use a profile
for a production deploy and a profile for test mode of your jobs.

Profiles are defined in the root `.conf` file in the `profiles` section. Any
profile should have a name and a description. Profile name is a path delimited
by dots.

For example, if you have a profile named `duck.production`:

```
profiles: [
  { name: "duck.production", description: "" }
]
```

than you can define variables like this:

```
duck: {
  production: {
    calculation_start: "2010-06-12"
  }
}
```

When deploying you'll need to choose with which profile the project will be
deployed.

# Templates

Each file you put into your project is viewed by datamot as a
[mustache](https://mustache.github.io/) template. Datamot will render each file
using configurations files as models. And only after rendering and checking for
errors it'll try to deploy your jobs. Generally you can skip thinking about
templates and just use them as simple variables substitution. But if you want
you can create quite complex logic using

# Plugins

Plugins are the way to make your project laconic and reduce code duplication.

Lets look at them by a simple yet very practical example, by creating an import
job. Merely any import job contains at least two files: `coordinator.xml` and
`workflow.xml`. But for a typical import job all we need to specify is an
output, connection string and a query:

```
output: hdfs:///path/to/out/dataset
connection: rdbms:///some/connection/string
query: "select * from joy"
```

Not that much, right? But, again, any time we need to create a scheduled import,
we need to create and fill at least that described two files: coordinator, in
which we specify output dataset and a workflow in which we create a sqoop
action. Let's move all them into a plugin.

```
  .git
  ...
  .plugins/
    import/
      coordinator.xml
      workflow.xml
  ...
```

Now, when we have a plugin for imports, to create a new import job we only need
to create a single file, with only three variables in it:

```
  .git
  .plugins/
  ...
  jobs/
    import-user-stats/
      import-user-stats.plug
  ...
```

The plug-file itself looks like this:

```
run: ".plugins/import"
with: {
  production: {
    output: "hdfs:///path/to/out/dataset"
    connection: "rdbms:///some/connection/string"
    query: "select * from joy"
  }
}
```

where `production` is a profile name. It's necessary to specify values for
variables used in a plugin for each profile declared in the root `.conf`.

Deploying the project will lead to the generation of corresponding coordinator
and workflow for this import.

# Autodeploy

Datamot can automatically deploy a project to a cluster on a push event.
Before deploying, it'll check the project for errors. Currently most typical
errors like referring to unexistent datasets, xml syntax errors, oozie syntax
errors are catched. The deploy will not be held in case of errors.

Any failures and compilation errors are shown in the output of the git push
command. If everything is ok, than datamot will look at `.deploy.conf` file in
the project root. Here is how it generally looks:

```
deploy: {
  oozie: {
    autodeploy : "production"
    coordinators: {
      "import-happiness": "kill,start"
      "calculate-joy": "kill,start"
    }
    done: true
    errors: []
  }
}
```

Datamot will perform automatic deploy on push only if `done` flag is set to
`false` and `autodeploy` is set to one of the declared profiles. Deploy consists
of the following actions:

1. Substitute variables and check for errors
2. If ok, upload the whole project to HDFS
3. For each cooridnator declared in the `deploy.oozie.coordinators` section
   execute listed commands. Commands are executed in the specified order.
   Order by which each coordinator is processed is not defined.

