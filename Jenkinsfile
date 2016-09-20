def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

node {
    // Checkout source code from Git
    stage 'Checkout'
    checkout scm

    // Build Docker image
    stage 'Build'
    sh "docker build -t throwaway123321/velocity-dcos:${gitCommit()} ."

    // Log in and push image to GitLab
    stage 'Publish'
    withCredentials(
        [[
            $class: 'UsernamePasswordMultiBinding',
            credentialsId: 'dockerhub-meaningful-throwaway',
            passwordVariable: 'DOCKERHUB_PASSWORD',
            usernameVariable: 'DOCKERHUB_USERNAME'
        ]]
    ) {
        sh "docker login -u ${env.DOCKERHUB_USERNAME} -p ${env.DOCKERHUB_PASSWORD} -e demo@mesosphere.com"
        sh "docker push throwaway123321/velocity-dcos:${gitCommit()}"
    }
}

// Deploy
stage 'Deploy'

marathon(
    url: 'http://marathon.mesos:8080',
    forceUpdate: false,
    credentialsId: 'dockerhub-meaningful-throwaway',
    filename: 'marathon.json',
    appId: 'dockerhub-meaningful-throwaway',
    docker: "throwaway123321/velocity-dcos:${gitCommit()}".toString()
)
