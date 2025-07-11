<?php
$host = "localhost";
$user = "root";
$pass = "";
$db = "69secreto";

$conn = new mysqli($host, $user, $pass, $db);
if ($conn->connect_error) {
    die("Erro: " . $conn->connect_error);
}
?> <?php session_start(); ?>
<h2>Login</h2>
<form action="login.php" method="post">
  <input name="email" placeholder="E-mail" required>
  <input name="senha" type="password" placeholder="Senha" required>
  <button type="submit">Entrar</button>
</form>
<a href="register.php">Cadastre-se</a> <?php
include("config.php");

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $nome = $_POST["nome"];
    $email = $_POST["email"];
    $senha = password_hash($_POST["senha"], PASSWORD_DEFAULT);
    $tipo = "pro";
    $hoje = date("Y-m-d H:i:s");
    $expira = date("Y-m-d H:i:s", strtotime("+1 day"));

    $stmt = $conn->prepare("INSERT INTO usuarios (nome, email, senha, tipo, data_pro_expira, usou_teste) VALUES (?, ?, ?, ?, ?, 1)");
    $stmt->bind_param("sssss", $nome, $email, $senha, $tipo, $expira);
    $stmt->execute();
    header("Location: index.php");
}
?>
<h2>Cadastro</h2>
<form method="post">
  <input name="nome" placeholder="Nome" required>
  <input name="email" placeholder="E-mail" required>
  <input name="senha" type="password" placeholder="Senha" required>
  <button type="submit">Cadastrar</button>
</form> <?php
session_start();
include("config.php");

$email = $_POST["email"];
$senha = $_POST["senha"];

$sql = "SELECT * FROM usuarios WHERE email='$email'";
$result = $conn->query($sql);

if ($result->num_rows == 1) {
    $user = $result->fetch_assoc();
    if (password_verify($senha, $user["senha"])) {
        $_SESSION["user_id"] = $user["id"];
        header("Location: feed.php");
        exit();
    }
}
echo "Login inválido."; <?php
session_start();
include("config.php");

$id = $_SESSION["user_id"];
$u = $conn->query("SELECT * FROM usuarios WHERE id=$id")->fetch_assoc();
$agora = date("Y-m-d H:i:s");

if ($u['tipo'] == 'pro' && $u['data_pro_expira'] < $agora) {
    $conn->query("UPDATE usuarios SET tipo='gratis' WHERE id=$id");
    $u['tipo'] = 'gratis';
}

$res = $conn->query("SELECT * FROM postagens ORDER BY destaque DESC, id DESC");
?>
<h2>Feed</h2>
<a href="upload.php">Nova Postagem</a>
<?php if ($u['tipo'] == 'gratis'): ?>
  <p style="color:red;">Sua conta é grátis. Assine para liberar todas funções.</p>
<?php endif; ?>
<?php while ($row = $res->fetch_assoc()): ?>
  <div>
    <img src="uploads/<?= $row['imagem'] ?>" width="300"><br>
    <strong><?= $row['descricao'] ?></strong>
    <?= $row['destaque'] ? '<span style="color:gold;">★ Destaque</span>' : '' ?>
  </div>
<?php endwhile; ?> <?php
session_start();
include("config.php");

$id = $_SESSION["user_id"];
$u = $conn->query("SELECT * FROM usuarios WHERE id=$id")->fetch_assoc();
$contar = $conn->query("SELECT COUNT(*) as total FROM postagens WHERE usuario_id=$id")->fetch_assoc();

$limite = ($u['tipo'] == 'pro') ? 999 : 5;

if ($contar['total'] >= $limite) die("Limite de postagens atingido.");

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $desc = $_POST["descricao"];
    $img = $_FILES["imagem"]["name"];
    move_uploaded_file($_FILES["imagem"]["tmp_name"], "uploads/$img");
    $conn->query("INSERT INTO postagens (usuario_id, imagem, descricao) VALUES ($id, '$img', '$desc')");
    header("Location: feed.php");
}
?>
<form method="post" enctype="multipart/form-data">
  <input name="descricao" required>
  <input type="file" name="imagem" required>
  <button type="submit">Postar</button>
</form> <?php
session_start();
include("../config.php");

$de = $_SESSION["user_id"];
$para = $_GET["para"];

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $msg = $_POST["mensagem"];
    $conn->query("INSERT INTO mensagens (de_id, para_id, texto) VALUES ($de, $para, '$msg')");
}

$msgs = $conn->query("SELECT * FROM mensagens WHERE (de_id=$de AND para_id=$para) OR (de_id=$para AND para_id=$de) ORDER BY id ASC");
?>
<h2>Chat</h2>
<?php while ($m = $msgs->fetch_assoc()): ?>
  <p><strong><?= $m['de_id'] == $de ? "Você" : "Outro" ?>:</strong> <?= $m['texto'] ?></p>
<?php endwhile; ?>
<form method="post">
  <input name="mensagem" required>
  <button type="submit">Enviar</button>
</form> <?php
include("../config.php");

$res = $conn->query("SELECT * FROM usuarios");
echo "<h2>Usuários</h2>";
while ($u = $res->fetch_assoc()) {
    echo "<p>{$u['nome']} - {$u['email']} - {$u['tipo']} - 
        <a href='tornar_pro.php?id={$u['id']}'>[+ PRO]</a> | 
        <a href='remover.php?id={$u['id']}'>[Excluir]</a></p>";
}
?> CREATE DATABASE 69secreto;
USE 69secreto;

CREATE TABLE usuarios (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(100),
  email VARCHAR(100),
  senha VARCHAR(255),
  tipo VARCHAR(10) DEFAULT 'gratis',
  cidade VARCHAR(100),
  data_pro_expira DATETIME,
  usou_teste BOOLEAN DEFAULT 0
);

CREATE TABLE postagens (
  id INT AUTO_INCREMENT PRIMARY KEY,
  usuario_id INT,
  imagem VARCHAR(255),
  descricao TEXT,
  destaque BOOLEAN DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);

CREATE TABLE mensagens (
  id INT AUTO_INCREMENT PRIMARY KEY,
  de_id INT,
  para_id INT,
  texto TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
