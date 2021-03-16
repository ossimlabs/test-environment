properties([
    parameters([
        string(name: 'PROJECT_URL', defaultValue: 'https://github.com/ossimlabs/Test-Environment', description: 'The project github URL'),
        string(name: 'DOCKER_REGISTRY_DOWNLOAD_URL', defaultValue: 'nexus-docker-private-group.ossim.io', description: 'Repository of docker images'),
        string(name: 'TESTS_TO_RUN', defaultValue: 'ALL', description: 'Used to specify which tests to run, default is all, runs all tests'),
        string(name: 'TEST_ENV', defaultValue: 'omar-test.ossim', description: 'Change this value to change the testing environment, i.e. change to omar-dev to test dev')
    ]),
    pipelineTriggers([
        [$class: "GitHubPushTrigger"]
    ]),
    [$class: 'GithubProjectProperty', displayName: '', projectUrlStr: '${PROJECT_URL}'],
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '3', daysToKeepStr: '', numToKeepStr: '20')),
    disableConcurrentBuilds()
    ])

podTemplate(
    containers: [
        containerTemplate(
            name: 'cypress',
            image: "${DOCKER_REGISTRY_DOWNLOAD_URL}/cypress/included:4.9.0",
            ttyEnabled: true,
            command: 'cat',
            privileged: true
        )
      ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        ),
    ]
)

{
    node(POD_LABEL)
    {
        // stage("test1")
        // {
        //     container('cypress')
        //     {
        //         sh """
        //         ls
        //         pwd
        //         """
        //         dir('dir1')
        //         {
        //             git url: 'https://github.com/ossimlabs/omar-wfs.git'
        //         }
        //         dir('dir2')
        //         {
        //             git url: 'https://github.com/ossimlabs/omar-wmts.git'
        //         }
        //         dir('dir3')
        //         {
        //             git url: 'https://github.com/ossimlabs/omar-oldmar.git'
        //         }
        //         sh """
        //             ls
        //             echo 'test 1 ...'
        //             cd dir1
        //             ls
        //             pwd
        //         """
        //         try {
        //             sh """
        //                 cypress run --headless
        //             """
        //         }
        //         catch (err) {}
        //     }
        // }
        stage("test2")
        {
            // git url: 'https://github.com/ossimlabs/omar-wfs.git'
            container('cypress')
            {
                if (TESTS_TO_RUN == 'ALL' || TESTS_TO_RUN == 'sqs-stager')
                {
                    stage('Run sqs-stager test')
                    {
                        sh """
                        git clone https://github.com/ossimlabs/omar-sqs-stager.git
                        cp omar-sqs-stager/cypress.json .
                        cp -r omar-sqs-stager/cypress .


                        sed 's+omar.ossim+${TEST_ENV}+g' cypress.json
                        sed 's+omar-dev.ossim+${TEST_ENV}+g' cypress.json
                        sed 's+omar-test.ossim+${TEST_ENV}+g' cypress.json


                        ls
                        """
                        try {
                        sh """
                            cypress run --headless
                        """
                        }
                        catch (err) {}
                        sh """
                            npm i -g xunit-viewer
                            xunit-viewer -r results -o results/omar-sqs-stager-test-results.html
                        """
                            junit 'results/*.xml'
                            archiveArtifacts "results/*.xml"
                            archiveArtifacts "results/*.html"
                            s3Upload(file:'results/omar-sqs-stager-test-results.html', bucket:'ossimlabs', path:'cypressTests/')
                        sh """
                        rm cypress.json
                        rm -r cypress
                        """
                    }
                }
                if (TESTS_TO_RUN == 'ALL' || TESTS_TO_RUN == 'oldmar')
                {
                    stage('Run oldmar test')
                    {
                        sh """
                        git clone https://github.com/ossimlabs/omar-oldmar.git
                        cp omar-oldmar/cypress.json .
                        cp -r omar-oldmar/cypress .
                        """
                        try {
                        sh """
                            cypress run --headless
                        """
                        }
                        catch (err) {}
                        sh """
                            npm i -g xunit-viewer
                            xunit-viewer -r results -o results/omar-oldmar-test-results.html
                        """
                            junit 'results/*.xml'
                            archiveArtifacts "results/*.xml"
                            archiveArtifacts "results/*.html"
                            s3Upload(file:'results/omar-oldmar-test-results.html', bucket:'ossimlabs', path:'cypressTests/')
                    }
                }
            }
        }
    }
}