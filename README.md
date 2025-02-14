app.py
from flask import Flask, request, jsonify, render_template
import pandas as pd

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return jsonify({'error': 'No file part'})

    file = request.files['file']
    if file.filename == '':
        return jsonify({'error': 'No selected file'})

    try:
        df = pd.read_excel(file)
        return jsonify({'columns': df.columns.tolist(), 'data': df.to_dict(orient='records')})
    except Exception as e:
        return jsonify({'error': str(e)})

if __name__ == '__main__':
    app.run(debug=True)

templates/index.html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Excel 解析器</title>
    <script>
        function uploadFile() {
            let formData = new FormData();
            let fileInput = document.getElementById("file");
            if (fileInput.files.length === 0) {
                alert("請選擇檔案！");
                return;
            }

            formData.append("file", fileInput.files[0]);

            fetch("/upload", { method: "POST", body: formData })
                .then(response => response.json())
                .then(data => {
                    if (data.error) {
                        alert("錯誤：" + data.error);
                        return;
                    }
                    displayTable(data.columns, data.data);
                })
                .catch(error => console.error("錯誤:", error));
        }

        function displayTable(columns, data) {
            let table = "<table border='1'><tr>";
            columns.forEach(col => table += `<th>${col}</th>`);
            table += "</tr>";

            data.forEach(row => {
                table += "<tr>";
                columns.forEach(col => table += `<td>${row[col] || ''}</td>`);
                table += "</tr>";
            });

            table += "</table>";
            document.getElementById("output").innerHTML = table;
        }
    </script>
</head>
<body>
    <h2>上傳 Excel 並顯示表格</h2>
    <input type="file" id="file">
    <button onclick="uploadFile()">上傳</button>
    <div id="output"></div>
</body>
</html>
