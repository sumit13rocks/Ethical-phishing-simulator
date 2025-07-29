# Ethical-phishing-simulator
from flask import Flask, render_template, request, redirect from flask_sqlalchemy import SQLAlchemy from datetime import datetime import os

app = Flask(name) app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///phishing.db' app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

class TargetInteraction(db.Model): id = db.Column(db.Integer, primary_key=True) email = db.Column(db.String(100)) action = db.Column(db.String(50)) timestamp = db.Column(db.DateTime, default=datetime.utcnow)

@app.route('/') def index(): return render_template('phishing_template.html')

@app.route('/click/<email>') def click(email): log = TargetInteraction(email=email, action="clicked") db.session.add(log) db.session.commit() return redirect('/login-form')

@app.route('/login-form', methods=['GET', 'POST']) def login_form(): if request.method == 'POST': email = request.form.get('email') password = request.form.get('password') log = TargetInteraction(email=email, action=f"submitted: {password}") db.session.add(log) db.session.commit() return redirect('/training') return render_template('login_form.html')

@app.route('/training') def training(): return render_template('training.html')

@app.route('/admin') def admin(): logs = TargetInteraction.query.order_by(TargetInteraction.timestamp.desc()).all() return render_template('admin.html', logs=logs)

if name == 'main': if not os.path.exists('phishing.db'): with app.app_context(): db.create_all() app.run(debug=True)
