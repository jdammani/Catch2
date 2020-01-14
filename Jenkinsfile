#!groovy


def buildStartTime = ""
def committerUserName = ""
def commitId = ""

def CleanUp(location){
	echo "Removing directory - " + location
	//bat "rmdir /s /Q $location"
}


def GetUserName (commitHash)
	{
		// Git committer User id
		echo "WorkSpacePath is ${WORKSPACE}"
		sh "cd ${WORKSPACE};git show -s --format='%an' ${commitHash} > commandResult"
		result = readFile('commandResult').trim()
		result = result.replace(" ", "").replace("," ,"")
		if (result.contains("\\"))
			{
			result = result.split("\\\\")[1]		
			}
		//echo "Git committer email: ${result}"
		return result

	}	
	

pipeline
	{
		agent { node { label 'dev' } }
		//agent any
		stages
			{
			// clean up
			stage('cleanup at Beginning') 
				{
				steps
					{
						cleanWs()		 
					        //CleanUp('%workspace%')
					}
				}

			// Source code checkout
			stage('Checkout SourceCode') 
				{
				steps
					{					
					echo 'Pulling...' + env.BRANCH_NAME
					script 
						{						
							def now = new Date()
							buildStartTime = now.format("yyyy-MM-dd'T'HH:mm:ss.SSS+00:00", TimeZone.getTimeZone("UTC")) 
							println "starttime:" + buildStartTime
																
							commitId = checkout(scm).GIT_COMMIT //checkout scm
							echo 'Commit id is : ' + commitId

							committerUserName = GetUserName(commitId)
							echo 'UserName:'+committerUserName
						}										
					}
				}							

			// build
			stage('Build') 
				{
				steps
					{
					script 
						{		
              						echo "Building Catch2"
							sh "echo $PWD"
							//sh "cd ${WORKSPACE}/src/;bash ./build-hworld.sh ${WORKSPACE}/src/"
              sh "cd ${WORKSPACE}/CMake;cmake -Bbuild -H. -DBUILD_TESTING=OFF"
							sh "zip -r build.zip build"
              echo "completed build"	              
						}
					}
				}
				
			// upload
			stage('Upload to S3 repository') 
				{
				steps
					{
					script 
						{		
              echo "uploading build.zip to cicd-buildpoc-repo/repository/Catch2/1.0/${env.BUILD_NUMBER}/"
							sh "aws s3 cp ${WORKSPACE}/build.zip s3://cicd-buildpoc-repo/repository/Catch/1.0/${env.BUILD_NUMBER}/build.zip"
							echo "upload successful"													
						}
					}
				}								
			}    	
	}					
