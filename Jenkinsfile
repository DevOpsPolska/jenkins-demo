    stage 'CI'
    
    node {
        git branch: 'master', 
        url: 'https://github.com/awslabs/opsworks-demo-php-simple-app.git'
        // pull dependencies from npm
        sh 'npm install'
        // stash code &amp; in temporary archive
        stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
        // test with PhantomJS for "fast" "generic" results
        // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
        sh 'npm run test-single-run -- --browsers PhantomJS'
    
        // archive karma test results (karma is configured to export junit xml files)
        step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
          
    }

    // demoing a second agent - add a node label if you spin up a second agent to ensure this runs on another agent.
    node {
        sh 'ls'
        sh 'rm -rf *'
        unstash 'everything'
        sh 'ls'
    }

    //parallel integration testing

    stage 'Browser Testing'
    parallel chrome: {
        runTests("Chrome")
    }, firefox: {
        runTests("Firefox")
    }, safari: {
        runTests("Safari")
    }

    def runTests(browser) {
        node {
            // on windows use: bat 'del /S /Q *'
            sh 'rm -rf *'
            unstash 'everything'
            sh "npm run test-single-run -- --browsers ${browser}"
            step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
        }
    }

    node {
        notify("Deploy to staging?")
    }

    input 'Deploy to staging?'

    // limit concurrency so we don't perform simultaneous deploys
    // and if multiple pipelines are executing, 
    // newest is only that will be allowed through, rest will be canceled
    stage name: 'Deploy', concurrency: 1
    node {
        // write build number to index page so we can see this update
        sh "echo '&lt;h1&gt;${env.BUILD_DISPLAY_NAME}&lt;/h1&gt;' &gt;&gt; app/index.html"
    
        // deploy to a docker container mapped to port 3000
        // on windows use: bat 'docker-compose up -d --build'
        sh 'docker-compose up -d --build'
    
        notify 'Solitaire Deployed!'
    }

    def notify(status){
        emailext (
            to: "wesmdemos@gmail.com",
            subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
            body: """&lt;p&gt;${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':&lt;/p&gt;
            &lt;p&gt;Check console output at &lt;a href='${env.BUILD_URL}'&gt;${env.JOB_NAME} [${env.BUILD_NUMBER}]&lt;/a&gt;&lt;/p&gt;""",
        )
    }