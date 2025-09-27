import string
import rando
from flask import Flask, request, redirect, render_template_string

app = Flask(__name__)
url_map = {}

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>URL Shortener</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { background: #f0f2f5; padding: 50px; }
        .card { max-width: 600px; margin: auto; border-radius: 15px; }
        input { border-radius: 10px; }
        button { border-radius: 10px; }
    </style>
</head>
<body>
    <div class="card shadow">
        <div class="card-body text-center">
            <h2 class="mb-4">🔗 URL Shortener</h2>
            <form method="post" class="d-flex mb-4">
                <input type="url" name="original_url" class="form-control me-2" placeholder="Enter your URL" required>
                <button type="submit" class="btn btn-primary">Shorten</button>
            </form>
            {% if short_url %}
                <div class="alert alert-success">
                    <strong>Short URL:</strong>
                    <a href="{{ short_url }}" target="_blank">{{ short_url }}</a>
                </div>
            {% endif %}
        </div>
    </div>
</body>
</html>
"""

def generate_short_id(length=6):
    characters = string.ascii_letters + string.digits
    return ''.join(random.choice(characters) for _ in range(length))

@app.route("/", methods=["GET", "POST"])
def home():
    short_url = None
    if request.method == "POST":
        original_url = request.form.get("original_url")
        if original_url:
            short_id = generate_short_id()
            url_map[short_id] = original_url
            short_url = request.host_url + short_id
    return render_template_string(HTML_TEMPLATE, short_url=short_url)

@app.route("/<short_id>")
def redirect_to_url(short_id):
    original_url = url_map.get(short_id)
    if original_url:
        return redirect(original_url)
    return "<h3>Invalid URL</h3>", 404

if __name__ == "__main__":
    app.run(debug=True)
