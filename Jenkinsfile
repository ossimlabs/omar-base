properties([
    parameters ([
        string(name: 'BUILD_NODE', defaultValue: 'omar-build', description: 'The build node to run on'),
        booleanParam(name: 'CLEAN_WORKSPACE', defaultValue: true, description: 'Clean the workspace at the end of the run')
    ]),
    pipelineTriggers([
            [$class: "GitHubPushTrigger"],
            pollSCM('H/2 * * * *')
    ])
])

node("${BUILD_NODE}"){

    stage("Checkout branch $BRANCH_NAME")
    {
        checkout(scm)
    }

    stage("Load Variables")
    {
        step ([$class: "CopyArtifact",
        projectName: "ossim-ci",
           filter: "common-variables.groovy",
           flatten: true])

        load "common-variables.groovy"
    }

    stage("Pull Artifacts")
    {
        step ([$class: "CopyArtifact",
            projectName: "ossim-ci",
            filter: "yum.repos.d"])

        step ([$class: "CopyArtifact",
            projectName: "ossim-ci",
            filter: "goofys",
            flatten: true])

    }    

    stage ("Publish Docker App")
    {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
                        credentialsId: 'dockerCredentials',
                        usernameVariable: 'DOCKER_REGISTRY_USERNAME',
                        passwordVariable: 'DOCKER_REGISTRY_PASSWORD']])
        {
            // Run all tasks on the app. This includes pushing to OpenShift and S3.
            sh """
            gradle pushDockerImage \
                -PossimMavenProxy=${OSSIM_MAVEN_PROXY}
            """
        }
    }
        
    stage("Clean Workspace")
    {
        if ("${CLEAN_WORKSPACE}" == "true")
            step([$class: 'WsCleanup'])
    }
}