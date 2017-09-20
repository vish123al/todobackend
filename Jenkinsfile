node {
    checkout scm

    try {
        stage 'Run unit/integration tests'
        sh 'curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose'
        sh 'chmod +x /usr/local/bin/docker-compose'
        sh 'make test'
        
        stage 'Build application artefacts'
        shell 'make build'

        stage 'Create release environment and run acceptance tests'
        shell 'make release'

        stage 'Tag and publish release image'
        shell "make tag latest \$(git rev-parse --short HEAD) \$(git tag --points-at HEAD)"
        shell "make buildtag master \$(git tag --points-at HEAD)"
        withEnv(["DOCKER_USER=${DOCKER_USER}",
                 "DOCKER_PASSWORD=${DOCKER_PASSWORD}",
                 "DOCKER_EMAIL=${DOCKER_EMAIL}"]) {    
            shell "make login"
        }
        shell "make publish"

        stage 'Deploy release'
        shell "printf \$(git rev-parse --short HEAD) > tag.tmp"
        def imageTag = readFile 'tag.tmp'
        build job: DEPLOY_JOB, parameters: [[
            $class: 'StringParameterValue',
            name: 'IMAGE_TAG',
            value: 'jmenga/todobackend:' + imageTag
        ]]
    }
    finally {
        stage 'Collect test reports'
        step([$class: 'JUnitResultArchiver', testResults: '**/reports/*.xml'])

        stage 'Clean up'
        shell 'make clean'
        shell 'make logout'
    }
}
