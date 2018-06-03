// Define your secret project token here
def project_token = 'abf102fa7f081d4b9727a8c52b88c2ce'

properties([
  [$class: 'GitLabConnectionProperty', gitLabConnection: 'gitlab'],
  pipelineTriggers([
      [
          $class: 'GitLabPushTrigger',
          branchFilterType: 'All',
          triggerOnPush: false,
          triggerOnMergeRequest: true,
          triggerOpenMergeRequestOnPush: "source",
          triggerOnNoteRequest: true,
          noteRegex: "Jenkins please retry a build",
          skipWorkInProgressMergeRequest: true,
          secretToken: project_token,
          ciSkip: false,
          setBuildDescription: true,
          addNoteOnMergeRequest: true,
          addCiMessage: true,
          addVoteOnMergeRequest: true,
          acceptMergeRequestOnSuccess: false,
          branchFilterType: "NameBasedFilter",
          includeBranchesSpec: "",
          excludeBranchesSpec: "",
      ]
  ]),
  buildDiscarder(
    logRotator(artifactDaysToKeepStr: '2', artifactNumToKeepStr: '8', daysToKeepStr: '2', numToKeepStr: '8')
  )
])

node() {
  try {
    // Pull the source and test a merge
    checkout changelog: true, poll: true, scm: [
      $class: 'GitSCM',
      branches: [[name: "origin/${env.gitlabSourceBranch}"]],
      doGenerateSubmoduleConfigurations: false,
      extensions: [[
        $class: 'PreBuildMerge',
        options: [
          fastForwardMode: 'FF',
          mergeRemote: 'origin',
          mergeStrategy: 'default',
          mergeTarget: "${env.gitlabTargetBranch}"
        ]
      ]],
      submoduleCfg: [],
      userRemoteConfigs: [[
        credentialsId: 'gitlab-jenkins-user-credentials',
        name: 'origin',
        url: "${env.gitlabSourceRepoHttpUrl}"
      ]]
    ]

    // Start the build
    load "Jenkinsfile.common"

  } catch (Exception e) {
    updateGitlabCommitStatus(name: 'build', state: 'failed')
    addGitLabMRComment comment: "Something unexpected happened. Inspect Jenkins logs."
    throw e
  }
}
