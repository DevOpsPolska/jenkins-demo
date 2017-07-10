node {

    git branch: ";jenkins2-course";, 
        url: ";https://github.com/g0t4/solitaire-systemjs-course";

    // pull dependencies from npm
    // on windows use: bat ";npm install";
    sh ";npm install";

    // stash code &amp; dependencies to expedite subsequent testing
    // and ensure same code &amp; dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: ";everything";, 
          excludes: ";test-results/**";, 
          includes: ";**";
    
    // test with PhantomJS for &quot;fast&quot; &quot;generic&quot; results
    // on windows use: bat ";npm run test-single-run -- --browsers PhantomJS";
    sh ";npm run test-single-run -- --browsers PhantomJS";
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: ";JUnitResultArchiver";, 
          testResults: ";test-results/**/test-results.xml";])
          
}

// demoing a second agent - add a node label if you spin up a second agent to ensure this runs on another agent.
node {

    // on windows use: bat ";ls";
    sh ";ls";

    // on windows use: bat ";del /S /Q *";
    sh ";rm -rf *";

    unstash ";everything";

    // on windows use: bat ";dir";
    sh ";ls";
}

//parallel integration testing
// on windows: swap out Safari below for IE or just use Chrome and Firefox
stage ";Browser Testing";
parallel chrome: {
    runTests(&quot;Chrome&quot;)
}, firefox: {
    runTests(&quot;Firefox&quot;)
}, safari: {
    runTests(&quot;Safari&quot;)
}

def runTests(browser) {
    node {
        // on windows use: bat ";del /S /Q *";
        sh ";rm -rf *";

        unstash ";everything";

        // on windows use: bat &quot;npm run test-single-run -- --browsers ${browser}&quot;
        sh &quot;npm run test-single-run -- --browsers ${browser}&quot;
        step([$class: ";JUnitResultArchiver";, 
              testResults: ";test-results/**/test-results.xml";])
    }
}

node {
    notify(&quot;Deploy to staging?&quot;)
}

input ";Deploy to staging?";

// limit concurrency so we don";t perform simultaneous deploys
// and if multiple pipelines are executing, 
// newest is only that will be allowed through, rest will be canceled
stage name: ";Deploy";, concurrency: 1
node {
    // write build number to index page so we can see this update
    
    // on windows use: bat &quot;echo ";&lt;h1&gt;${env.BUILD_DISPLAY_NAME}&lt;/h1&gt;"; &gt;&gt; app/index.html&quot;
    sh &quot;echo ";&lt;h1&gt;${env.BUILD_DISPLAY_NAME}&lt;/h1&gt;"; &gt;&gt; app/index.html&quot;
    
    // deploy to a docker container mapped to port 3000
    // on windows use: bat ";docker-compose up -d --build";
    sh ";docker-compose up -d --build";
    
    notify ";Solitaire Deployed!";
}











def notify(status){
    emailext (
      to: &quot;wesmdemos@gmail.com&quot;,
      subject: &quot;${status}: Job ";${env.JOB_NAME} [${env.BUILD_NUMBER}]";&quot;,
      body: &quot;&quot;&quot;&lt;p&gt;${status}: Job ";${env.JOB_NAME} [${env.BUILD_NUMBER}]";:&lt;/p&gt;
        &lt;p&gt;Check console output at &lt;a href=";${env.BUILD_URL}";&gt;${env.JOB_NAME} [${env.BUILD_NUMBER}]&lt;/a&gt;&lt;/p&gt;&quot;&quot;&quot;,
    )

