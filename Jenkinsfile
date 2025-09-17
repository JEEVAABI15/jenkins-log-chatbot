pipeline {
    agent any
    environment {
        FILE_PATH = "/var/lib/jenkins/jobs/previous_failed_builds.txt"
        EMAIL_RECIPIENTS = "shafeeqahamed1512@gmail.com, jeevaabishake@gmail.com"
        PYTHON_SCRIPT_PATH = "main.py"
        GEMINI_API_KEY = credentials('GEMINI_API_KEY')
        GEMINI_API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent"
        BASE_URL = "http://172.208.50.231"
    }
    stages {
        stage('Detect Newly Failed Build') {
            steps {
                script {
                    try {
                        echo "BASE URL: ${BASE_URL}"
                        
                        echo "Starting 'Detect Newly Failed Build' stage"
                        // Read the content of the file containing previous failed builds
                        def fileContent = fileExists(env.FILE_PATH) ? readFile(env.FILE_PATH) : ""
                        // Get the names of all active jobs
                        def activeJobs = getActiveJobNames()
                        echo "Active jobs: ${activeJobs}"
                        def newFailedBuilds = []

                        // Iterate over each active job
                        activeJobs.each { jobName ->
                            if (jobName != env.JOB_NAME) {
                                // Get the job object
                                def job = Jenkins.instance.getItem(jobName)
                                // Get all failed builds for the job
                                def failedBuilds = job.getBuilds().findAll { it.result == hudson.model.Result.FAILURE }
                                echo "Failed builds for job ${jobName}: ${failedBuilds}"

                                // Iterate over each failed build
                                failedBuilds.each { failedBuild ->
                                    def failureEntry = "${jobName}:${failedBuild.getNumber()}"
                                    // Check if the failed build is not already recorded
                                    if (!fileContent.contains(failureEntry)) {
                                        // Get the console output of the failed build
                                        def consoleOutput = failedBuild.getLog(Integer.MAX_VALUE).join('\n')
                                        consoleOutput = consoleOutput.replace('\n', '\\n')
                                        // Add the new failed build to the list
                                        newFailedBuilds << "${jobName}:${failedBuild.getNumber()}:${consoleOutput}"
                                    }
                                }
                            }
                        }
                        // Write the new failed builds to a file if any
                        if (newFailedBuilds) {
                            writeFile file: 'failed_builds.txt', text: newFailedBuilds.join("\n")
                        } else {
                            echo "No new failed builds to summarize"
                        }
                    } catch (Exception e) {
                        echo "Error finding new failures: ${e.message}"
                    }
                }
            }
        }
        stage('Summarize Failures & Send Email') {
            steps {
                script {
                    try {
                        echo "Starting 'Summarize Failures' stage"
                        // Read the new failed builds from the file
                        def newFailedBuilds = fileExists('failed_builds.txt') ? readFile('failed_builds.txt').split("\n") : 0
                        if (!newFailedBuilds) {
                            echo "No new failed builds to summarize. Skipping Python and Gemini API call."
                        } else {
                            // Iterate over each new failed build
                            newFailedBuilds.each { failure ->
                                def details = failure.split(":",3)
                                def jobName = details[0]
                                def failID = details[1]
                                def consoleLog = details[2]
                                echo "Processing new failed builds - Job Name: ${jobName}, Build ID: ${failID}"

                                if (consoleLog?.trim()) {
                                    echo "Sending log to Gemini API for summarization..."
                                    
                                    // Properly escape the console log to handle special characters
                                    def escapedConsoleLog = consoleLog.replace("\n", "\\n").replace("\"", "\\\"")

                                    // Initialize variables
                                    def errorType = "Unknown Error"
                                    def summary = "Failed to retrieve summary."
                                    // uniqueKey already declared above

                                    // Read the Python script from main.py
                                    def pythonScript = readFile(env.PYTHON_SCRIPT_PATH)

                                    def result = ""
                                    try {
                                        echo "Running Python script to summarize the log..."
                                        result = sh(
                                            script: """python3 "${env.PYTHON_SCRIPT_PATH}" --value "${escapedConsoleLog}" --api_key "${env.GEMINI_API_KEY}" --api_url "${env.GEMINI_API_URL}" """,
                                            returnStdout: true
                                        ).trim()
                                        echo "Python script execution complete. Result length: ${result.length()}"
                                    } catch (Exception e) {
                                        echo "Error executing Python script: ${e.message}"
                                        result = "Error executing API summarization."
                                    }

                                    def jsonPayload = groovy.json.JsonOutput.toJson([
                                        job_name: jobName,
                                        build_number: failID,
                                        log: consoleLog
                                    ])
                                    writeFile file: 'payload.json', text: jsonPayload
                                    
                                    // Declare uniqueKey variable at higher scope to avoid redeclaration
                                    def uniqueKey = "unknown"
                                    
                                    def response = sh(script: """
                                        curl -X POST '${BASE_URL}:8000/chatbot/load' \\
                                        -H 'accept: application/json' \\
                                        -H 'Content-Type: application/json' \\
                                        -d @payload.json
                                    """, returnStdout: true).trim()
                                    
                                    // Write response to a file for safe parsing
                                    writeFile file: 'response.json', text: response
                                    echo "API Response: ${response}"
                

                                    try {
                                        // Parse the response as JSON properly
                                        def responseJson = readJSON text: response
                                        echo "Successfully parsed API response JSON: ${response}"
                                        
                                        // Extract unique_key from the response
                                        if (responseJson.containsKey('unique_key')) {
                                            uniqueKey = responseJson.unique_key
                                            echo "Found field with name 'unique_key'"
                                        } else {
                                            // Handle the case where the field might have a different name
                                            echo "Field 'unique_key' not found in response"
                                            
                                            // Extract from URL in the response message if available
                                            def responseMessage = responseJson.response ?: ""
                                            def urlMatcher = responseMessage =~ /\/chatbot\/([a-zA-Z0-9]+)/
                                            
                                            if (urlMatcher.find()) {
                                                uniqueKey = urlMatcher[0][1]
                                                echo "Extracted unique_key from URL: ${uniqueKey}"
                                            } else {
                                                uniqueKey = "unknown"
                                                echo "Could not extract unique_key from response"
                                            }
                                        }
                                        
                                        echo "Final unique_key: ${uniqueKey}"
                                    } catch (Exception e) {
                                        echo "Error parsing API response JSON: ${e.message}"
                                        echo "Raw response content: ${response}"
                                        uniqueKey = "unknown"
                                    }

                                    def parsedResult = [:]
                                    try {
                                        echo "Attempting to parse result: ${result}"
                                        parsedResult = readJSON text: result
                                        echo "Successfully parsed Gemini API result: ${parsedResult}"
                                    } catch (Exception e) {
                                        echo "Error parsing Gemini API result: ${e.message}"
                                        echo "Raw result content: ${result}"
                                    }
                                    
                                    if (parsedResult?.errorType && parsedResult?.summary) {
                                        errorType = parsedResult.errorType
                                        summary = parsedResult.summary
                                    } else {
                                        errorType = "Unknown Error"
                                        summary = "Failed to retrieve summary."
                                    }

                                    echo "Summarization completed"

                                    echo "Error Type: ${errorType}"
                                    echo "Summary: ${summary}"
                                    echo "Unique Key: ${uniqueKey}"
                                    
                                    // Safe chat URL construction
                                    def chatURL = "${BASE_URL}"
                                    if (uniqueKey && uniqueKey != "unknown") {
                                        chatURL = "${BASE_URL}?chat=${uniqueKey}"
                                        echo "Chat URL: ${chatURL}"
                                    } else {
                                        echo "WARNING: Unable to construct chat URL due to missing uniqueKey"
                                    }

                                    // Construct URLs for the job and console
                                    def jobURL = "${Jenkins.instance.getRootUrl()}job/${jobName}/${failID}"
                                    def consoleURL = "${Jenkins.instance.getRootUrl()}job/${jobName}/${failID}/console"
                                    
                                    // Construct the email body
                                    def chatSection = ""
                                    if (uniqueKey && uniqueKey != "unknown") {
                                        chatSection = "<p><b>Continue chat:</b> <a href=\"${chatURL}\">${chatURL}</a></p>"
                                        echo "Added chat section with URL: ${chatURL}"
                                    } else {
                                        echo "Skipped adding chat section due to missing uniqueKey"
                                    }
                                    
                                    def emailBody = """
                                        <p>Dear All,</p>
                                        <p>A new failed build has been detected:</p>
                                        <p><b>Job Name:</b> ${jobName}</p>
                                        <p><b>Build Number:</b> ${failID}</p>
                                        <p><b>Job URL:</b> <a href="${jobURL}">${jobURL}</a></p>
                                        <p><b>Console URL:</b> <a href="${consoleURL}">${consoleURL}</a></p>
                                        <p><b>Error Type:</b> ${errorType}</p>
                                        <p><b>Summary: </b> ${summary}</p>
                                        ${chatSection}
                                        <p>Regards,</p>
                                        <p>Jenkins Admin</p>
                                    """
                                    // Send the email notification
                                    mail to: env.EMAIL_RECIPIENTS,
                                         subject: "New Failed Build on ${jobName} ERROR: ${errorType}",
                                         body: emailBody,
                                         mimeType: 'text/html'

                                } else {
                                    echo "Skipping summarization: Empty or invalid console log."
                                }
                            }
                        }
                    } catch (Exception e) {
                        echo "Error during summarization: ${e.message}"
                    }
                }
            }
        }
        stage('Cleanup') {
            steps {
                script {
                    try {
                        echo "Starting 'Cleanup' stage"
                        // Read the new failed builds from the file
                        def newFailedBuilds = fileExists('failed_builds.txt') ? readFile('failed_builds.txt').split("\n") : []
                        if (newFailedBuilds) {
                            // Read the content of the file containing previous failed builds
                            def fileContent = fileExists(env.FILE_PATH) ? readFile(env.FILE_PATH) : ""
                            // Append the new failed builds to the file
                            writeFile file: env.FILE_PATH, text: fileContent + "\n" + newFailedBuilds.join("\n")
                            echo "File updated with new failed builds."
                            // Remove the temporary file containing new failed builds
                            sh "rm -rf failed_builds.txt"
                        }
                    } catch (Exception e) {
                        echo "Error during cleanup: ${e.message}"
                    }
                }
            }
        }
    }
}

// Helper functions
@NonCPS
def getActiveJobNames() {
    echo "Fetching active job names"
    // Get all active jobs
    def activeJobs = Jenkins.instance.getAllItems(hudson.model.Job.class)
    // Return the names of the active jobs
    return activeJobs.collect { it.name }
}
