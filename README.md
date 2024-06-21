FPTKA C07

![image](https://github.com/ch0clat/FPTKA/assets/128571877/63d02ffb-55c1-44f8-b978-0762cc4ae038)

![image](https://github.com/ch0clat/FPTKA/assets/128571877/2449dcf7-1cc0-49aa-a7d9-0cfa7f6b94dc)

![image](https://github.com/ch0clat/FPTKA/assets/128571877/960f7c96-a9e6-49ef-82a2-73c0c838ff12)



```
server {
    listen 80;
    server_name 146.190.99.144; #IP VM

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

``` py
import os
from flask import Flask, request, jsonify
from flask_cors import CORS
from textblob import TextBlob
from pymongo import MongoClient

app = Flask(__name__)
CORS(app)

# Setting MonggoDB
username = os.environ.get('MONGODB_USERNAME', 'doadmin')
password = os.environ.get('MONGODB_PASSWORD', 'vY75Z3n2b8omR609')
host = os.environ.get('MONGODB_HOST', 'mongodb+srv://db-mongodb-sgp1-33856-dde5f5ff.mongo.ondigitaloce>
database = os.environ.get('MONGODB_DATABASE', 'admin')

# Database setup
client = MongoClient(f'mongodb+srv://{username}:{password}@{host}/{database}?retryWrites=true&w=majori>
db = client.sentiment_analysis
collection = db.history

@app.route('/analyze', methods=['POST'])
def analyze_sentiment():
    data = request.get_json()
    text = data.get('text', '')
    analysis = TextBlob(text)
    sentiment = analysis.sentiment.polarity

    # Save to database
    collection.insert_one({'text': text, 'sentiment': sentiment})

    return jsonify({'sentiment': sentiment})

@app.route('/history', methods=['GET'])
def get_history():
    history = list(collection.find({}, {'_id': 0}).sort("_id", -1))
    return jsonify(history)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

![image](https://github.com/ch0clat/FPTKA/assets/128571877/336faa64-00e2-4a4f-966a-a9d205906a62)
