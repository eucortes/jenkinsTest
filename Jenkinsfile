
pipeline 
{
    agent { 
		node {
			label "W12_SBX_Bld" 
			customWorkspace  "C:\\j\\FLMMB\\${BUILD_NUMBER}"

		}
	}

	environment {
		msBuildTools = addQuotes("${tool 'MSBuild_2019_Enterprise'}")
    }
    
    stages 
    {
		stage("identify Branch")
		{
			when { anyOf{branch "master"; branch "main"; branch "release/*"  }}
			steps{
				script{branchType="release"}
			}
		}
		stage("identify Branch Step 2")
		{
			when { not{ anyOf{branch "master"; branch "main"; branch "release/*"  }}}
			steps{
				script{branchType="feature"}
			}
		}
		stage("Naming drop folder")
		{
			steps
			{
				script{
				BaseDropFolder ="${BaseDropFolder}${branchType}"
				GitDropfolder="C:${BaseDropFolder}"
				}
			}
		}
        stage ("Cleanup") 
        {
			steps 
			{
				bat """
					echo "Performing Stage Start Tasks"
					echo "Setup and clean current repo"
					git reset --hard
					git clean -fdx
				"""
			}
		}

		stage ('NugetRestore')
		{
			echo "nuget"
		}
    stage ("Build sbX-FloorManager") 
    {
      steps {
        echo "Build sbX-FloorManager..."
      }
    }
		stage ('NugetPush')
		{
			steps {
           echo "push nuget"
			
			}
		}
		stage('Prune Old DropFolders')
		{
			steps{
				echo 'Prune'
			}
		}

	}
	post {
		success {
			script{
				emailext body: 'Job Name: ${JOB_NAME}\nBuild URL: ${BUILD_URL}\nLog: ${BUILD_URL}console', 
					recipientProviders: [culprits()], 
					subject: 'Build Success: [$BUILD_STATUS] #$BUILD_NUMBER - $PROJECT_NAME'
			}
			script{
				emailext attachLog: true, compressLog: true, mimeType: 'text/html',
				body: ' \\\\${NODE_NAME}\\C\$${BaseDropFolder}\\${BuildPrefix} \n${JELLY_SCRIPT, template="html"}', 
				recipientProviders: [culprits()], 
				subject: 'Build Failed: [$BUILD_STATUS] #$BUILD_NUMBER - $PROJECT_NAME'
			}
		}	

		failure {
			script {
				emailext attachLog: true, compressLog: true, mimeType: 'text/html',
				body: '${JELLY_SCRIPT, template="html"}', 
				recipientProviders: [culprits()], 
				subject: 'Build Failed: [$BUILD_STATUS] #$BUILD_NUMBER - $PROJECT_NAME'
				
			}
		}
		always{
			cleanWs(cleanWhenFailure: false, 
			cleanWhenNotBuilt: false,
			cleanWhenAborted: false)
			echo "Built at ${NODE_NAME} ${GitDropfolder}\\${BuildPrefix}..."
			echo "\\\\${NODE_NAME}\\C\$${BaseDropFolder}\\"
		}
	}	

    // The options directive is for configuration that applies to the whole job.
    options 
    {
        // For example, we'd like to make sure we only keep X builds at a time, so we don't fill up our storage!
        buildDiscarder(logRotator(numToKeepStr:'2'))

        // And we'd really like to be sure that this build doesn't hang forever, so let's time it out
        timeout(time: 360, unit: 'MINUTES')

        // Add timestamps to the output
        timestamps()

        // Disable concurrent builds
        disableConcurrentBuilds()
    }
}
