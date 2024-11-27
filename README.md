from flask import Flask, render_template_string, request
import yt_dlp
import os

app = Flask(__name__)

# Carpeta de salida para los archivos descargados
output_dir = "descargas"
os.makedirs(output_dir, exist_ok=True)

# Plantilla HTML directamente en el archivo
html_template = """
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Descargar Música y Videos</title>
    <style>
        body {
            background-color: black;
            color: white;
            font-family: Arial, sans-serif;
        }
        h1 {
            text-align: center;
        }
        .container {
            width: 50%;
            margin: 0 auto;
            padding-top: 50px;
        }
        .form-group {
            margin-bottom: 20px;
        }
        input[type="text"], select {
            width: 100%;
            padding: 10px;
            margin-top: 5px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        button {
            background-color: #1DB954;
            color: white;
            border: none;
            padding: 10px 20px;
            cursor: pointer;
            border-radius: 5px;
        }
        button:hover {
            background-color: #1ed760;
        }
        .error {
            color: red;
            text-align: center;
        }
        .success {
            color: green;
            text-align: center;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>🎶 Descarga Música y Videos 📹</h1>

        {% if error %}
        <p class="error">{{ error }}</p>
        {% endif %}

        {% if success %}
        <p class="success">{{ success }}</p>
        {% endif %}

        <form method="POST">
            <div class="form-group">
                <label for="url">URL del Video o Canción</label>
                <input type="text" id="url" name="url" placeholder="Introduce la URL" required>
            </div>

            <div class="form-group">
                <label for="tipo">Selecciona el tipo de descarga</label>
                <select id="tipo" name="tipo">
                    <option value="Música">Música</option>
                    <option value="Video">Video</option>
                </select>
            </div>

            <button type="submit">Descargar</button>
        </form>
    </div>

</body>
</html>
"""

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        url = request.form.get("url")
        tipo = request.form.get("tipo")

        if not url.strip():
            return render_template_string(html_template, error="Por favor, introduce una URL válida.")
        
        try:
            # Opciones de yt-dlp
            ydl_opts = {}
            if tipo == "Música":
                ydl_opts = {
                    'format': 'bestaudio/best',
                    'outtmpl': os.path.join(output_dir, '%(title)s.%(ext)s'),
                    'postprocessors': [{
                        'key': 'FFmpegExtractAudio',
                        'preferredcodec': 'mp3',
                        'preferredquality': '192',
                    }],
                }
            elif tipo == "Video":
                ydl_opts = {
                    'format': 'bestvideo+bestaudio',
                    'outtmpl': os.path.join(output_dir, '%(title)s.%(ext)s'),
                }

            with yt_dlp.YoutubeDL(ydl_opts) as ydl:
                ydl.download([url])

            return render_template_string(html_template, success=f"{tipo} descargado con éxito en la carpeta '{output_dir}'!")
        except Exception as e:
            return render_template_string(html_template, error=f"Error al procesar la URL: {e}")
    
    return render_template_string(html_template)

if __name__ == "__main__":
    app.run(debug=True)
