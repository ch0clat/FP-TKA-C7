<div align=center>

# Laporan Pengerjaan - Final Project TKA

</div>

    
# Kelompok C7
    
### Fikri Aulia As Sa'adi - 5027231026

### Rama Owarianto Putra Suharjito - 50272310249

### Abhirama Triadyatma Hermawan  - 5027231061

### Hasan - 5027231073

### Nabiel Nizar Anwari - 5027231087

# Permasalahan

Anda adalah seorang lulusan Teknologi Informasi, sebagai ahli IT, salah satu kemampuan yang harus dimiliki adalah Keampuan merancang, membangun, mengelola aplikasi berbasis komputer menggunakan layanan awan untuk memenuhi kebutuhan organisasi.

Pada suatu saat anda mendapatkan project untuk mendeploy sebuah aplikasi Sentiment Analysis dengan komponen Backend menggunakan python: sentiment-analysis.py dengan spesifikasi sebagai berikut

# Rincian Harga VM

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

## Instalasi
1. Install nginx `sudo apt install nginx`
2. Pengubahan ip pada `index.html`
``` html
<script>
        document.getElementById('sentiment-form').addEventListener('submit', async function(event) {
            event.preventDefault();
            const text = document.getElementById('text-input').value;
            try {
                const response = await fetch('http://146.190.99.144:5000/analyze', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({ text }),
                });
                if (!response.ok) throw new Error('Network response was not ok');
                const result = await response.json();
                const resultElement = document.getElementById('result');
                resultElement.textContent = `Sentiment Score: ${result.sentiment}`;
                resultElement.className = result.sentiment > 0 ? 'positive' : 'negative';
                fetchHistory();
            } catch (error) {
                console.error('There has been a problem with your fetch operation:', error);
            }
        });

        async function fetchHistory() {
            try {
                const response = await fetch('http://146.190.99.144:5000/history');
                if (!response.ok) throw new Error('Network response was not ok');
                const history = await response.json();
                const historyList = document.getElementById('history');
                historyList.innerHTML = '';
                history.forEach(item => {
                    const listItem = document.createElement('li');
                    listItem.className = `list-group-item history-item ${item.sentiment > 0 ? 'positiv>
                    listItem.textContent = `Text: ${item.text}, Sentiment: ${item.sentiment}`;
                    historyList.appendChild(listItem);
                });
            } catch (error) {
                console.error('There has been a problem with your fetch operation:', error);
            }
        }

        // Fetch history on page load
        fetchHistory();
    </script>
```
3. Memindahkan `index.html` ke dalam directory `/var/www/html`
4. Config nginx
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
4. `sudo systemctl restart nginx`
5. Insialisasi virtual environments `python3 -m venv venv`
6. Instalasi requirments `pip3 install flask flask-cors textblob pymongo`
7. Konfigurasi MonggodDB
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
8. Instalasi gunicorn `pip install gunicorn`
9. Run Flask App dengan gunicorn `gunicorn -w 4 -b 0.0.0.0:5000 sentiment-analysis:app`

![image](https://github.com/ch0clat/FPTKA/assets/128571877/336faa64-00e2-4a4f-966a-a9d205906a62)

## API & Interface

![image](https://github.com/ch0clat/FPTKA/assets/128571877/3e49ac8e-df91-4ca9-bf6c-e946a8064e75)

![image](https://github.com/ch0clat/FPTKA/assets/128571877/13d99568-9453-46a7-bffb-7e0c76456509)

![image](https://github.com/ch0clat/FPTKA/assets/128571877/e8f77a2e-e68d-48d9-bb32-c16eafd4086e)

## Locust

!(github-small)[https://github.com/bielnzar/TKAC7.github.io/blob/main/1_user.png]

## Kesimpulan

1. Kami telah berhasil men-deploy aplikasi Sentiment Analysis dengan komponen backend menggunakan Python.
Infrastruktur yang digunakan meliputi:

2. Kami menggunakan dua VM worker (VM1 dan VM2) untuk menangani permintaan aplikasi, database MongoDB untuk penyimpanan data, Load Balancer untuk mendistribusikan beban kerja

3. Alat yang kami digunakan mencakup:
    - Nginx sebagai web server
    - Flask sebagai framework Python untuk backend
    - Gunicorn sebagai WSGI HTTP Server
    - TextBlob untuk analisis sentimen
    - PyMongo untuk koneksi ke MongoDB

4. Aplikasi berhasil dijalankan dan dapat melakukan analisis sentimen serta menyimpan riwayat analisis.

