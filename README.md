# Cylc Conda

Conda recipe to install the *cylc-8.0a1* "alpha-1" preview release of Cylc 8.

Includes:
- Python 3.7
- cylc-flow-8.0a1 - Python 3 Workflow Service and CLI
- cylc-uiserver-0.1 - Python 3 UI Server component of the Cylc 8 architecture
- cylc-ui-0.1 - Vue.js web UI
- JupyterHub - authenticates users and launches their Cylc UI Servers
- configurable-http-proxy - node.js proxy
- (*and all software dependencies of the above*)

The Cylc 8 Architecture is described
[here](https://cylc.github.io/cylc-admin/cylc-8-architecture.html)

## Current limitations

**Cylc-8.0a1 is an early full-system Cylc 8 preview release**
- It has a fully functional Python 3 workflow service and CLI that can run
  cylc-7 workflows

**But:**
- It is not production-ready yet
  - Use the latest cylc-7.8 release in production
- Do not use it where security is a concern
- The UI only includes our prototype "workflow tree view"
  - with no control capability yet (just monitoring)
  - we are working on several other views, include a dependency graph view
- Data update in the UI is via polling at 5 second intervals, and monolithic
  - future releases will use WebSockets and incremental update
- There is no capability to start workflows from the web UI yet

## Instructions

Enable the `kinow` Conda channel in your `.condarc` file (pending package
availability on conda-forge):

```
channels:
  - defaults
  - conda-forge
  - kinow
ssl_verify: true
auto_activate_base: false
anaconda_upload: false
```

Create a new Conda environment, e.g. `cylc1`.

- `conda create -n cylc1`
- `conda activate cylc1`

Install Cylc

- `conda install cylc`

You should now be able to run Cylc commands, e.g.:

- `cylc --version`
- `cylc run --no-detach my.suite`
- `cylc-uiserver --help`

Start the hub (jupyterhub gets installed with the "cylc" package):

- `jupyterhub
    --JupyterHub.spawner_class="jupyterhub.spawner.LocalProcessSpawner"
    --Spawner.args="['-s', '${CONDA_PREFIX}/work/cylc-ui']"
    --Spawner.cmd="cylc-uiserver"
    --JupyterHub.logo_file="${CONDA_PREFIX}/work/cylc-ui/img/logo.png"`

Voilà 🎉. Go to `http://localhost:8000`, log in wto the Hub ith your local user
credentials, and enjoy Cylc 8 Alpha-1!

- Start a workflow with the CLI (a good example is shown below)
- Log in at the Hub to authenticate and launch your UI Server
- Note that much of the UI Dashboard is not functional yet.
  The functional links are:
  - Cylc Hub
  - Suite Design Guide (web link)
  - Documentation (web link)
- In the left side-bar, click on Workflows to see a list of running workflows
- In Workflows view, click on an icon under "Actions" to view the
  corresponding workflow. 
- In the tree view:
  - click on task names to see the list of task jobs
  - click on job icons to see the detail of a specific job

To deactivate and/or remove the conda environment:

- `conda deactivate`
- `conda env remove -n cylc1`

## An Example Workflow to View

The following workflow generates a bunch of tasks that initially fail before
succeeding after a random number of retries (this shows the new "Cylc 8
task/job separation" nicely):

```
[cylc]
   cycle point format = %Y
   [[parameters]]
      m = 0..5
      n = 0..2
[scheduling]
   initial cycle point = 3000
   [[graph]]
      P1Y = "foo[-P1Y] => foo => bar<m> => qux<m,n> => waz"
[runtime]
   [[root]]
      script = """
sleep 10
# fail 50% of the time if try number is less than 5
if (( CYLC_TASK_TRY_NUMBER < 5 )); then
  if (( RANDOM % 4 < 2 )); then
     exit 1
  fi
fi"""
      [[[job]]]
         execution retry delays = 6*PT2S
```
