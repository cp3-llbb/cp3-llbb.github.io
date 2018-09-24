# Framework Jenkins CI

When opening a new Pull Request, an automated tool, [Jenkins](https://jenkins-ci.org/), takes care of launching a full build. It allows to see directly if your code can be merged without breaking everything. We have a dedicated Jenkins [instance](https://jenkins-cp3-llbb.web.cern.ch/) running at CERN. Only members of the ``cp3-llbb`` CERN e-group can access this page.

As soon as a new Pull Request is opened, or if an already opened Pull Request is updated, an automatic build is launched. Only one build can be executed at the same time: every other builds are queued. A build typically takes about 1 hour.

Once a build is started, the Pull Request status on GitHub is updated. Once done, the status will either be green (the code compiles) or red (something is wrong). You can click on the ``Details`` link to access the Jenkins job report and the compilation log. For more information, see [https://github.com/blog/1935-see-results-from-all-pull-request-status-checks](https://github.com/blog/1935-see-results-from-all-pull-request-status-checks).

The Pull Request won't be mergeable until the Pull Request status is green.

!!! important "Bootstrap"

    Since the build is automatized, Jenkins needs to know how-to setup the CMSSW env by itself. To do that, two files are necessary:

    - ``CMSSW.release``: This file must contains only a string representing the CMSSW version to use to setup the framework. Be careful not to add a line break at the end of the line.
    - `` CMSSW.arch``: The ``SCRAM_ARCH`` of the CMSSW release.
    - ``bootstrap_jenkins.sh``: This file is a bash script executed by Jenkins just before building the framework, but after the CMSSW env is setup. You **must** use this file to install all the dependencies of the framework.
    - ``jenkins_postbuild.sh``: This file is executed by Jenkins after the compilation.

    **Do not forget to update these files when changes are done to the release or the dependencies, otherwise the build will fail.**

## Technical details

 - Jenkins [instance](https://jenkins-cp3-llbb.web.cern.ch/)
 - [OpenShift](https://www.openshift.com/) instance: [jenkins-cp3-llbb](https://openshift.cern.ch/console/project/jenkins-cp3-llbb/overview) (access is restricted to administrators. If you want / need access, please ask Christophe D. or Sébastien B.). This is the platform hosting our Jenkins instance inside an isolated container (for more information, look for Docker on Google).

A [github bot](https://github.com/cp3-llbb-bot) also exists; it's a generic github user, member of the cp3-llbb organization. It needs push authorization to a repository to properly update the PR status. Password for this user can be found on the protected CP3 [wiki](https://cp3.irmp.ucl.ac.be/projects/cp3admin/wiki/UsersPage/Private/Physics/Exp/llbb)

!!! Note
    Sometimes, the container responsible for the build get stuck in creating phase and you'll need to kill it and retrigger a new build. To do that, connect to the OpenShift instance, and select on the left menu `Applications` → `Pods`. Click on the build instance in the list, and on the new page, select `Delete` in the `Actions` menu (top right). It may takes some time before the container is killed. If it's still stuck in deleting after a few minutes, you'll have to open a new ticket [here](https://cern.service-now.com/service-portal/service-element.do?name=PaaS-Web-App).

## Troubleshooting

If a build/test fails because of unexpected connection glitch, you can re-trigger jenkins by commenting `please test` to the pull request
