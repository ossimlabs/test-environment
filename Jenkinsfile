properties([
    parameters([
        string(name: 'PROJECT_URL', defaultValue: 'https://github.com/ossimlabs/test-environment', description: 'The project github URL'),
        string(name: 'DOCKER_REGISTRY_DOWNLOAD_URL', defaultValue: 'nexus-docker-private-group.ossim.io', description: 'Repository of docker images'),
        string(name: 'TESTS_TO_RUN', defaultValue: 'ALL', description: 'Used to specify which tests to run, default is all, runs all tests'),
        string(name: 'TEST_ENV', defaultValue: 'omar-test.ossim', description: 'Change this value to change the testing environment, i.e. change to omar-dev to test dev'),
        booleanParam(name: 'BACK_END', defaultValue: true, description: 'Should all the backEnd tests run?'),
        booleanParam(name: 'FRONT_END', defaultValue: true, description: 'Should all the frontEnd tests run?'),
        booleanParam(name: 'RUN_UI', defaultValue: true, description: 'Should UI test run?'),
        booleanParam(name: 'RUN_TLV', defaultValue: true, description: 'Should tlv test run?')
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
            image: "${DOCKER_REGISTRY_DOWNLOAD_URL}/cypress/included:4.9.2",
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
        stage("Checkout branch")
        {
            scmVars = checkout(scm)
            Date date = new Date()
            String currentDate = date.format("MM/dd-HH:mm")
            if (params.BACK_END && params.FRONT_END)
            {
                buildName "${TESTS_TO_RUN}-${currentDate}"
                TAG_NAME = "${TESTS_TO_RUN}-${currentDate}"
            }
            else if (!params.FRONT_END && params.BACK_END)
            {
                if (TESTS_TO_RUN == 'ALL')
                {
                    buildName "All_BackEnd" + "-${currentDate}"
                    TAG_NAME = "All_BackEnd" + "-${currentDate}"
                }
                else
                {
                    buildName "${TESTS_TO_RUN}-${currentDate}"
                    TAG_NAME = "${TESTS_TO_RUN}-${currentDate}"
                }
            }
            else if (params.FRONT_END && !params.BACK_END)
            {
                if (TESTS_TO_RUN == 'ALL')
                {
                    buildName "All_FrontEnd" + "-${currentDate}"
                    TAG_NAME = "All_FrontEnd" + "-${currentDate}"
                }
                else if (!params.RUN_UI && params.RUN_TLV)
                {
                    buildName "TLV" + "-${currentDate}"
                    TAG_NAME = "TLV" + "-${currentDate}"
                }
                else if (params.RUN_UI && !params.RUN_TLV)
                {
                    buildName "UI" + "-${currentDate}"
                    TAG_NAME = "UI" + "-${currentDate}"
                }
            }
        }




        if (params.BACK_END)
        {
            stage("Begin BackEnd Tests")
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
        if (params.FRONT_END)
        {
            stage("Begin FrontEnd Tests")
            {
                if (params.RUN_UI)
                {
                    stage("UI test")
                    {
                        container('cypress')
                        {
                            sh """
                            git clone --branch jdk11 https://github.com/ossimlabs/omar-ui.git
                            cp omar-ui/cypress.json .
                            cp -r omar-ui/cypress .

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
                                xunit-viewer -r results -o results/omar-ui-test-results.html
                            """
                            junit 'results/*.xml'
                            archiveArtifacts "results/*.xml"
                            archiveArtifacts "results/*.html"
                            s3Upload(file:'results/omar-ui-test-results.html', bucket:'ossimlabs', path:'cypressTests/')

                            sh """
                            rm cypress.json
                            rm -r cypress
                            """
                        }
                    }
                }
                if (params.RUN_TLV)
                {
                    stage("TLV test")
                    {
                        container('cypress')
                        {
                            sh """
                                git clone https://github.com/ossimlabs/tlv.git
                                cp tlv/cypress.json .
                                cp -r tlv/cypress .
                                cp tlv/testParameters.json .

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
                                xunit-viewer -r results -o results/tlv-test-results.html
                                """
                                junit 'results/*.xml'
                                archiveArtifacts "results/*.xml"
                                archiveArtifacts "results/*.html"
                                s3Upload(file:'results/tlv-test-results.html', bucket:'ossimlabs', path:'cypressTests/')

                            sh """
                                cd cypress/jsonFiles
                                chmod +x fixCypressOutput.sh
                                ./fixCypressOutput.sh

                                cd ../testing/testing/spiders
                                scrapy crawl tests -o output.json
                                chmod +x fixScrapyOutput.sh
                                ./fixScrapyOutput.sh

                                cd ../../..

                                python3 comparison.py

                                cd ..
                            """
                            try
                            {
                              sh """
                                  cypress run --headless --spec "cypress/integration/Final.js"
                              """
                            }
                            catch (err) {}
                            sh """
                               npm i -g xunit-viewer
                               xunit-viewer -r results -o results/tlv-test-results.html
                            """
                            junit 'results/*.xml'
                            archiveArtifacts "results/*.xml"
                            archiveArtifacts "results/*.html"
                            s3Upload(file:'results/tlv-test-results.html', bucket:'ossimlabs', path:'cypressTests/')

                            sh """
                            rm cypress.json
                            rm -r cypress
                            rm testParameters.json
                            """
                        }
                    }
                }
            }
        }
    }
}