@Library('Jenkins-shared-libs') _

pipeline {
    agent any

    parameters {
        choice(
            name: 'HOST_FOR_DEPLOY',
            choices: ['do17.dev.kimver.devops.rebrain.srwx.net', 'do17.prod.kimver.devops.rebrain.srwx.net'],
            description: ''
        )
        gitParameter branchFilter: 'origin/(.*)', defaultValue: 'master', name: 'BRANCH', type: 'PT_BRANCH', description: 'Git branch selection'
    }	

    options {
        gitLabConnection('gitlab')
        gitlabBuilds(builds: ['jenkins'])
    }

    triggers {
        gitlab(
                triggerOnPush: true,
                branchFilterType: 'All',
                secretToken: 'rx-uB68r85hXxfA-MCEm'
        )
    }

    post {
        failure {
            updateGitlabCommitStatus name: 'jenkins', state: 'failed'
        }
        success {
            updateGitlabCommitStatus name: 'jenkins', state: 'success'
        }
    }
	
    stages {
        stage('Prepair') {
            steps {
		            git branch: 'master',
                url: 'https://gitlab.rebrainme.com/kimver/do17.git',
		            credentialsId: 'gitlab_cred'
            }
	      }
        stage('Build') {
            steps {
                npmBuild()
            }
	          post {
		            success {
            	      archiveArtifacts artifacts: 'build/'
		            }
	          }
	      }
	      stage('Deploy') {
	          steps {
		            sh label: '', script: '''
		            dst_dir="/var/www/$HOST_FOR_DEPLOY/$BUILD_NUMBER"
		            mkdir -p $dst_dir
		            cp -rf build $dst_dir/
		            ln -sfn $dst_dir/build /etc/nginx/$HOST_FOR_DEPLOY-latest
		            '''
	          }
	      }
    }
}
