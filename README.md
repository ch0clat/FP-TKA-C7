<div align=center>

# Laporan Pengerjaan - Final Project TKA

</div>

    
# Kelompok C7
    
### Fikri Aulia As Sa'adi - 5027231026

### Rama Owarianto Putra Suharjito - 50272310249

### Abhirama Triadyatma Hermawan  - 5027231061

### Hasan - 5027231073

### Nabiel Nizar Anwari - 5027231087

# a. Rincian Harga VM

### VM1 : WORKER 1
Spesifikasi : 2GB Memory, 1 AMD vCPU, 50GB Disk NVMe SSD, dan 2 TB transfer
Biaya : 14$
Berperan sebagai salah satu app worker yang menangani permintaan aplikasi dari load balancer.

![Screenshot 2024-06-14 110229](https://github.com/ch0clat/FPTKA/assets/142889150/f59fbad8-57e1-4d04-89b9-512d3272dabf)


### VM2 : WORKER 2
Spesifikasi : 1GB Memory, 1 Intel vCPU, 35GB Disk NVMe SSD, dan 1 TB transfer
VM2 juga berfungsi sebagai app worker yang membantu memproses permintaan aplikasi dari load balancer.


![Screenshot 2024-06-14 110240](https://github.com/ch0clat/FPTKA/assets/142889150/0f41f03c-e37e-437e-a939-4b503104bb28)


### MongoDB
Spesifikasi :  1GB Memory, 1 vCPU, dan 15GB Disk NVMe SSD.
Digunakan sebagai database untuk menyimpan data yang diperlukan oleh aplikasi.

![Screenshot 2024-06-14 110157](https://github.com/ch0clat/FPTKA/assets/142889150/a437a9a9-8e0f-4095-85b1-74b4b08c7d9b)


### Load Balancing
Spesifikasi : Terdiri dari 2 nodes dan mampu menangani koneksi simultan hingga 1000, dengan kemampuan 10000 RPS (Requests Per Second) dan 250 SSL CPS (Connections Per Second).
Bertugas mendistribusikan permintaan dari pengguna ke VM1 dan VM2. Ini memastikan bahwa beban kerja dibagi secara merata

![Screenshot 2024-06-14 110123](https://github.com/ch0clat/FPTKA/assets/142889150/086010ed-f5df-4904-884c-87b173e140eb)


### Total Harga :
![Screenshot 2024-06-14 111808](https://github.com/ch0clat/FPTKA/assets/142889150/6db0002b-984f-4e7d-a920-e1889ab3be70)


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
