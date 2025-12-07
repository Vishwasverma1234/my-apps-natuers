pipeline {
  agent any

  environment {
    // nothing public here
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Frontend') {
      steps {
        dir('frontend') {
          // ensure npm modules installed
          sh 'npm ci'
          // Vite build -> dist/
          sh 'npm run build'
        }
      }
    }

    stage('Deploy Frontend to Netlify') {
      steps {
        // use Jenkins stored secret 'netlify-token' as NETLIFY_AUTH_TOKEN
        withCredentials([string(credentialsId: 'netlify-token', variable: 'NETLIFY_TOKEN')]) {
          dir('frontend') {
            // export token for netlify CLI then deploy production from dist/
            sh '''
              export NETLIFY_AUTH_TOKEN=$NETLIFY_TOKEN
              # if your site is already linked in netlify CLI, use --prod --dir=dist
              # if not linked, use --site=SITE_ID (optional). Using --prod deploys to production
              netlify deploy --prod --dir=dist
            '''
          }
        }
      }
    }

    stage('Build Backend (optional)') {
      steps {
        dir('backend') {
          sh 'npm ci'
          // run tests or build steps for backend as needed
          // e.g. sh 'npm test' or build-step
        }
      }
    }

    stage('Trigger Render Deploy') {
      steps {
        // call the render deploy hook (stored in Jenkins)
        withCredentials([string(credentialsId: 'render-hook', variable: 'RENDER_HOOK')]) {
          sh '''
            # trigger the render deploy by POSTing to the hook URL
            curl -X POST "$RENDER_HOOK"
          '''
        }
      }
    }
  }

  post {
    success {
      echo 'Pipeline finished successfully'
    }
    failure {
      echo 'Pipeline failed'
    }
  }
}
