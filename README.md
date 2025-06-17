# TRABALHOLABAGENDA

from flask import Flask, render_template, request, redirect, url_for
import sqlite3

app = Flask(__name__)

# Função conexão
def conexao():
    conn = sqlite3.connect('database/labagenda.db')
    return conn


# Página inicial - lista agendamentos e usuários
@app.route('/')
def index():
    conn = conexao()
    cursor = conn.cursor()

    cursor.execute("SELECT * FROM agendamentos")
    agendamentos = cursor.fetchall()

    cursor.execute("SELECT * FROM usuarios")
    usuarios = cursor.fetchall()

    conn.close()
    return render_template('index.html', agendamentos=agendamentos, usuarios=usuarios)


# Rota para adicionar usuário
@app.route('/adicionar_usuario', methods=['POST'])
def adicionar_usuario():
    nome = request.form['nome']
    email = request.form['email']
    senha = request.form['senha']

    conn = conexao()
    cursor = conn.cursor()
    cursor.execute('''
        INSERT INTO usuarios (nome, email, senha) VALUES (?, ?, ?)
    ''', (nome, email, senha))
    conn.commit()
    conn.close()

    return redirect('/')


# Rota para excluir usuário
@app.route('/excluir_usuario/<int:id>')
def excluir_usuario(id):
    conn = conexao()
    cursor = conn.cursor()

    # Excluir agendamentos vinculados a esse usuário
    cursor.execute('DELETE FROM agendamentos WHERE usuario_id = ?', (id,))

    # Excluir o usuário
    cursor.execute('DELETE FROM usuarios WHERE id = ?', (id,))
    conn.commit()
    conn.close()

    return redirect('/')


# Rota para adicionar agendamento
@app.route('/adicionar_agendamento', methods=['POST'])
def adicionar_agendamento():
    laboratorio = request.form['laboratorio']
    data = request.form['data']
    horario = request.form['horario']
    responsavel = request.form['responsavel']
    usuario_id = request.form['usuario_id']

    conn = conexao()
    cursor = conn.cursor()
    cursor.execute('''
        INSERT INTO agendamentos (laboratorio, data, horario, responsavel, usuario_id)
        VALUES (?, ?, ?, ?, ?)
    ''', (laboratorio, data, horario, responsavel, usuario_id))
    conn.commit()
    conn.close()

    return redirect('/')
