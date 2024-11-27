from flask import Flask, request, redirect, url_for, render_template_string, session

app = Flask(__name__)
app.secret_key = "clave_secreta"  # Clave para sesiones

# Simulando base de datos en memoria
users = {"admin": "1234"}  # Usuarios: {usuario: contraseña}
posts = []  # Lista de publicaciones
friends = ["Alice", "Bob", "Charlie"]  # Lista de amigos

# Plantilla básica embebida
template = """
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .container { max-width: 600px; margin: auto; }
        .post { border: 1px solid #ccc; padding: 10px; margin: 10px 0; }
        .friends { background-color: #f9f9f9; padding: 10px; margin: 10px 0; }
        textarea, input { width: 100%; margin-bottom: 10px; }
    </style>
</head>
<body>
    <div class="container">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
"""

@app.route("/", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form.get("username")
        password = request.form.get("password")
        if username in users and users[username] == password:
            session["username"] = username
            return redirect(url_for("home"))
        return render_template_string(template, title="Iniciar Sesión", content="Usuario o contraseña incorrectos.")
    return render_template_string(template, title="Iniciar Sesión", content="""
        <h1>Iniciar Sesión</h1>
        <form method="POST">
            <input type="text" name="username" placeholder="Usuario" required>
            <input type="password" name="password" placeholder="Contraseña" required>
            <button type="submit">Ingresar</button>
        </form>
        <p>¿No tienes cuenta? <a href="{{ url_for('register') }}">Regístrate</a></p>
    """)

@app.route("/register", methods=["GET", "POST"])
def register():
    if request.method == "POST":
        username = request.form.get("username")
        password = request.form.get("password")
        if username in users:
            return render_template_string(template, title="Registro", content="El usuario ya existe.")
        users[username] = password
        return redirect(url_for("login"))
    return render_template_string(template, title="Registro", content="""
        <h1>Registro</h1>
        <form method="POST">
            <input type="text" name="username" placeholder="Usuario" required>
            <input type="password" name="password" placeholder="Contraseña" required>
            <button type="submit">Registrar</button>
        </form>
        <p>¿Ya tienes cuenta? <a href="{{ url_for('login') }}">Inicia sesión</a></p>
    """)

@app.route("/home", methods=["GET", "POST"])
def home():
    if "username" not in session:
        return redirect(url_for("login"))
    if request.method == "POST":
        content = request.form.get("content")
        if content:
            posts.append({"user": session["username"], "content": content})
    return render_template_string(template, title="Inicio", content=f"""
        <h1>Bienvenido, {session['username']}</h1>
        <a href="{{ url_for('logout') }}">Cerrar sesión</a>
        <h2>Publicar algo</h2>
        <form method="POST">
            <textarea name="content" placeholder="Escribe algo..."></textarea>
            <button type="submit">Publicar</button>
        </form>
        <div class="friends">
            <h3>Tus Amigos</h3>
            <ul>
                {''.join(f'<li>{friend}</li>' for friend in friends)}
            </ul>
        </div>
        <h3>Publicaciones</h3>
        {''.join(f'<div class="post"><b>{post["user"]}:</b> {post["content"]}</div>' for post in posts)}
    """)

@app.route("/logout")
def logout():
    session.pop("username", None)
    return redirect(url_for("login"))

if __name__ == "__main__":
    app.run(debug=True)
