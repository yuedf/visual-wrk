#!/usr/bin/env groovy          

pipeline {
	agent any
	options {                  
		buildDiscarder(logRotator(numToKeepStr: '25'))
	}
	parameters {               
		choice (               
				choices:'data/method_get.json\ndata/method_post.json\ndata/method_post_base64.json\ndata/mixed_test.json',
				description: 'The test case file used for testing',
				name : 'case_file')

			string (               
					defaultValue: '60',
					description: 'Number of Concurrent Connections',
					name : 'concurrency')

			string (
					defaultValue: '5m',
					description: 'Duration of test, time arguments may include a time unit (2s, 2m, 2h)',
					name : 'duration')

			string (
					defaultValue: '1',
					description: 'RPS Chart sample interval',
					name : 'sample_interval')

			string (
					defaultValue: 'https://navicore.mapbar.com',
					description: 'url(default read from json), example:https://navicore.mapbar.com',
					name : 'WRK_URL')
	}
	stages {
		stage('Test') { 
            echo 'making...'
            sh('make')
            sh('make install')
		}
		stage('Test') { 
			steps {
				echo 'testing...'           
					sh('visual-wrk -i${sample_interval} -c${concurrency} -d${duration} -j ${case_file} --latency')
			}
		}
		stage('Publish') {
			steps {
				sh('if [ ! -d "report/data" ]; then cd report && ln -s ../data/ data && cd -; fi;')
					publishHTML([allowMissing: true, 
							alwaysLinkToLastBuild: false, 
							keepAll: true, 
							reportDir: 'report', 
							reportFiles: 'log.html', 
							reportName: 'HTML Report', 
							reportTitles: ''])
			}
		}
	}
	post {
		failure {
			mail (body: "URL de build: <a href=\"${env.BUILD_URL}\">${env.BUILD_URL}</a>",
					charset: 'UTF-8',         
					mimeType: 'text/html',
					subject: "ERROR CI: ${env.JOB_NAME}",
					to: "zhaogq@mapbar.com")
		}
	}
}

