properties
([
    parameters
    ([
        string(name: 'PROJECT_URL', defaultValue: 'https://github.com/ossimlabs/test-environment', description: 'The project github URL'),
        string(name: 'DOCKER_REGISTRY_DOWNLOAD_URL', defaultValue: 'nexus-docker-private-group.ossim.io', description: 'Repository of docker images'),
        string(name: 'TESTS_TO_RUN', defaultValue: 'ALL', description: 'Used to specify which tests to run, default is all, runs all tests'),
        string(name: 'TEST_ENV', defaultValue: 'omar-test.ossim', description: 'Change this value to change the testing environment, i.e. change to omar-dev to test dev')
    ]),
    pipelineTriggers
    ([
        [$class: "GitHubPushTrigger"]
    ]),
    [$class: 'GithubProjectProperty', displayName: '', projectUrlStr: '${PROJECT_URL}'],
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '3', daysToKeepStr: '', numToKeepStr: '20')),
    disableConcurrentBuilds()
])

podTemplate
(
    containers:
    [
        containerTemplate
        (
            name: 'cypress',
            image: "${DOCKER_REGISTRY_DOWNLOAD_URL}/cypress/included:4.9.0",
            ttyEnabled: true,
            command: 'cat',
            privileged: true
        )
    ],
    volumes:
    [
        hostPathVolume
        (
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        ),
    ]
)

{
    node(POD_LABEL)
    {
        stage("Checkout branch")
        {
            //APP_NAME = PROJECT_URL.tokenize('/').last()
            scmVars = checkout(scm)
            Date date = new Date()
            String currentDate = date.format("MM/dd-HH:mm")
            //MASTER = "master"
            //GIT_BRANCH_NAME = scmVars.GIT_BRANCH
            //BRANCH_NAME = """${sh(returnStdout: true, script: "echo ${GIT_BRANCH_NAME} | awk -F'/' '{print \$2}'").trim()}"""
            //VERSION = """${sh(returnStdout: true, script: "cat chart/Chart.yaml | grep version: | awk -F'version:' '{print \$2}'").trim()}"""
            //VERSION = 1.0
            //GIT_TAG_NAME = APP_NAME + "-" + VERSION

            buildName "${TESTS_TO_RUN}-${currentDate}"
            TAG_NAME = "${TESTS_TO_RUN}-${currentDate}"
        }

        stage("Begin Tests")
        {
            def fileContents = readFile "testList.txt"
            def loop = 0
            def go = 0
            String[] str
            str = fileContents.split('\n')

            if (TESTS_TO_RUN == 'ALL')
            {
                loop = str.size()
            }
            else
            {
                loop = 1
                str = [TESTS_TO_RUN]
            }
            while(go < loop)
            {
                for( String currTest : str )
                {
                    print currTest
                    container('cypress')
                    {
                        stage(currTest + " test")
                        {
                            sh """
                            git clone https://github.com/ossimlabs/omar-"$currTest".git
                            cp omar-"$currTest"/cypress.json .
                            cp -r omar-"$currTest"/cypress .

                            sed 's+omar.ossim+${TEST_ENV}+g' cypress.json > cypress2.json && cat cypress2.json > cypress.json && rm cypress2.json
                            sed 's+omar-dev.ossim+${TEST_ENV}+g' cypress.json > cypress2.json && cat cypress2.json > cypress.json && rm cypress2.json
                            sed 's+omar-test.ossim+${TEST_ENV}+g' cypress.json > cypress2.json && cat cypress2.json > cypress.json && rm cypress2.json
                            """

                            try
                            {
                                sh """
                                    cypress run --headless
                                """
                            }
                            catch (err) {}


                            sh """
                                npm i -g xunit-viewer
                                xunit-viewer -r results -o results/omar-"$currTest"-test-results.html
                            """
                                junit 'results/*.xml'
                                archiveArtifacts "results/*.xml"
                                archiveArtifacts "results/*.html"
                                s3Upload(file:'results/omar-' + "${currTest}" + '-test-results.html', bucket:'ossimlabs', path:'cypressTests/')
                            sh """
                                rm cypress.json
                                rm -r cypress
                            """
                        }
                    }
                    go++
                }
            }
        }
    }
}