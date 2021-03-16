properties([
    parameters([
        string(name: 'PROJECT_URL', defaultValue: 'https://github.com/ossimlabs/Test-Environment', description: 'The project github URL'),
        string(name: 'DOCKER_REGISTRY_DOWNLOAD_URL', defaultValue: 'nexus-docker-private-group.ossim.io', description: 'Repository of docker images')
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
node(POD_LABEL){
    stage("Checkout branch") {
        container('cypress') {
            try {
                sh """
                    cypress run --headless
                """
            }
            catch (err) {
            }
                sh """
                    npm i -g xunit-viewer
                    xunit-viewer -r results -o results/${APP_NAME}-test-results.html
                """
                    junit 'results/*.xml'
                    archiveArtifacts "results/*.xml"
                    archiveArtifacts "results/*.html"
                    s3Upload(file:'results/${APP_NAME}-test-results.html', bucket:'ossimlabs', path:'cypressTests/')
                }
    }

    stage("Load Variables") {}

    stage('SonarQube Analysis') {}

    stage('Build') {}

    stage('Docker Build') {}

    stage('Docker Push') {}

    stage('Package & Upload Chart'){}

    stage('Tag Repo') {}
    }
}