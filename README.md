### Architecture Diagram
![image](https://github.com/user-attachments/assets/0062bcdd-8067-46a2-96c6-9cc678d041ab)

### Sample Stored Data
```json
{
  "1dbf59287172": {
    "job_name": "Job 1",
    "build_number": "16",
    "log": "Started by user admin\n[Pipeline] Start of Pipeline\n[Pipeline] node\nRunning on Jenkins in /var/lib/jenkins/workspace/Job1\n[Pipeline] {\n[Pipeline] stage\n[Pipeline] { (Check Parameter)\n[Pipeline] script\n[Pipeline] {\n[Pipeline] error\n[Pipeline] }\n[Pipeline] // script\n[Pipeline] }\n[Pipeline] // stage\n[Pipeline] stage\n[Pipeline] { (Declarative: Post Actions)\n[Pipeline] echo\nJob failed. Please review your input.\n[Pipeline] }\n[Pipeline] // stage\n[Pipeline] }\n[Pipeline] // node\n[Pipeline] End of Pipeline\nERROR: Build failed due to user input.\nFinished: FAILURE",
    "memory": "User: Added Logs\nLOGS:Started by user admin\n[Pipeline] Start of Pipeline\n[Pipeline] node\nRunning on Jenkins in /var/lib/jenkins/workspace/Job1\n[Pipeline] {\n[Pipeline] stage\n[Pipeline] { (Check Parameter)\n[Pipeline] script\n[Pipeline] {\n[Pipeline] error\n[Pipeline] }\n[Pipeline] // script\n[Pipeline] }\n[Pipeline] // stage\n[Pipeline] stage\n[Pipeline] { (Declarative: Post Actions)\n[Pipeline] echo\nJob failed. Please review your input.\n[Pipeline] }\n[Pipeline] // stage\n[Pipeline] }\n[Pipeline] // node\n[Pipeline] End of Pipeline\nERROR: Build failed due to user input.\nFinished: FAILURE\nUser: Why the job failed\nGemini: The Jenkins job failed because of an error within the \"Check Parameter\" stage.  The log indicates invalid user input caused the failure. Review the parameter definitions and input validation in your pipeline script.\n\nUser: pipeline {\n    agent any\n    parameters {\n         choice(name: 'RESULT',\n               choices: ['fail', 'pass'],\n               description: 'Select \"pass\" to succeed or \"fail\" to fail.')\n    }\n    stages {\n        stage('Check Parameter') {\n            steps {\n                script {\n                    if (params.RESULT == 'pass') {\n                        echo \"Build succeeded!\"\n                    } else if (params.RESULT == 'fail') {\n                        error(\"Build failed due to user input.\")\n                    } else {\n                        error(\"Invalid input. Please enter 'pass' or 'fail'.\")\n                    }\n                }\n            }\n        }\n    }\n    post {\n        success {\n            echo \"Job completed successfully!\"\n        }\n        failure {\n            echo \"Job failed. Please review your input.\"\n        }\n    }\n}\nGemini: The provided pipeline correctly handles the 'pass' and 'fail' choices.  The failure is due to the user selecting neither option, triggering the \"Invalid input\" error.  Ensure the user only selects 'pass' or 'fail' in the job parameters.\n\nUser: yf\nGemini: No logs were provided.  Please provide logs for analysis or ask a Jenkins-related question.\n",
    "history": {
      "1738491646": {
        "user": "LOG ADDED\nStarted by user admin\n[Pipeline] Start of Pipeline\n[Pipeline] node\nRunning on Jenkins in /var/lib/jenkins/workspace/Job1\n[Pipeline] {\n[Pipeline] stage\n[Pipeline] { (Check Parameter)\n[Pipeline] script\n[Pipeline] {\n[Pipeline] error\n[Pipeline] }\n[Pipeline] // script\n[Pipeline] }\n[Pipeline] // stage\n[Pipeline] stage\n[Pipeline] { (Declarative: Post Actions)\n[Pipeline] echo\nJob failed. Please review your input.\n[Pipeline] }\n[Pipeline] // stage\n[Pipeline] }\n[Pipeline] // node\n[Pipeline] End of Pipeline\nERROR: Build failed due to user input.\nFinished: FAILURE",
        "gemini": "Thanks for giving the log. Please ask your question"
      },
      "1738491672": {
        "user": "Why the job failed",
        "gemini": "The Jenkins job failed because of an error within the \"Check Parameter\" stage.  The log indicates invalid user input caused the failure. Review the parameter definitions and input validation in your pipeline script.\n"
      },
      "1738491687": {
        "user": "pipeline {\n    agent any\n    parameters {\n         choice(name: 'RESULT',\n               choices: ['fail', 'pass'],\n               description: 'Select \"pass\" to succeed or \"fail\" to fail.')\n    }\n    stages {\n        stage('Check Parameter') {\n            steps {\n                script {\n                    if (params.RESULT == 'pass') {\n                        echo \"Build succeeded!\"\n                    } else if (params.RESULT == 'fail') {\n                        error(\"Build failed due to user input.\")\n                    } else {\n                        error(\"Invalid input. Please enter 'pass' or 'fail'.\")\n                    }\n                }\n            }\n        }\n    }\n    post {\n        success {\n            echo \"Job completed successfully!\"\n        }\n        failure {\n            echo \"Job failed. Please review your input.\"\n        }\n    }\n}",
        "gemini": "The provided pipeline correctly handles the 'pass' and 'fail' choices.  The failure is due to the user selecting neither option, triggering the \"Invalid input\" error.  Ensure the user only selects 'pass' or 'fail' in the job parameters.\n"
      },
      "1738491717": {
        "user": "yf",
        "gemini": "No logs were provided.  Please provide logs for analysis or ask a Jenkins-related question.\n"
      }
    }
  }
}
```

### Setup Commands

```sh
cd frontend
npm i
npm start
npm run build
cp -r dist/* /var/www/html/
sudo systemctl restart nginx
```

```sh
cd backend
python3 -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload
sudo systemctl restart nginx
sudo vi /etc/systemd/system/fastapi.service
sudo systemctl enable fastapi
sudo systemctl start fastapi
sudo systemctl status fastapi
```

### ENV File Config
```
LOGS_DATA_FILE="Data_FILE"
GEMINI_API_KEY="API_KEY"
GEMINI_MODEL_NAME="modelVersion:generationMethod"
```
