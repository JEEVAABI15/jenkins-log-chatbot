### Architecture Diagram
![image](https://github.com/user-attachments/assets/0062bcdd-8067-46a2-96c6-9cc678d041ab)


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
