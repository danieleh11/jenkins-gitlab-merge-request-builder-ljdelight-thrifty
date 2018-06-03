properties([
  [$class: 'GitLabConnectionProperty', gitLabConnection: 'gitlab'],
  buildDiscarder(
    logRotator(artifactDaysToKeepStr: '2', artifactNumToKeepStr: '8', daysToKeepStr: '2', numToKeepStr: '8')
  )
])
triggers {
    gitlab(
            triggerOnPush: false,
            triggerOnMergeRequest: true,
            triggerOpenMergeRequestOnPush: "source",
            triggerOnNoteRequest: true,
            noteRegex: "Jenkins please retry a build",
            skipWorkInProgressMergeRequest: false,
            ciSkip: false,
            setBuildDescription: false,
            addNoteOnMergeRequest: false,
            addCiMessage: false,
            addVoteOnMergeRequest: false,
            acceptMergeRequestOnSuccess: false,
            branchFilterType: "NameBasedFilter",
            includeBranchesSpec: "${BRANCH_NAME}",
            secretToken: "abf102fa7f081d4b9727a8c52b88c2ce"
    )
}
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
