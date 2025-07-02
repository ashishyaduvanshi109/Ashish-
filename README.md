ashok-enterprises/
├── index.html              ← Customer form
├── upload.php              ← Save form data
├── admin-login.php         ← Admin login page
├── admin-check.php         ← Login authentication
├── admin.php               ← Admin dashboard
├── logout.php              ← Admin logout
├── /uploads/               ← Uploaded files
└── /uploads/.htaccess      ← Folder protection# <!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Ashok Enterprises Form</title>
  <style>
    body { font-family: Arial; background: #f2f2f2; }
    .container {
      max-width: 500px;
      margin: 50px auto;
      background: white;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.2);
    }
    input, label, button {
      display: block;
      width: 100%;
      margin-top: 10px;
    }
    button {
      background: red;
      color: white;
      padding: 10px;
      border: none;
      margin-top: 20px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>Ashok Enterprises</h2>
    <form action="upload.php" method="POST" enctype="multipart/form-data">
      <label>Name</label>
      <input type="text" name="name" required />
      <label>Phone</label>
      <input type="tel" name="phone" required pattern="[0-9]{10}" />
      <label>Email</label>
      <input type="email" name="email" required />
      <label>Upload Document</label>
      <input type="file" name="document" accept=".pdf,.jpg,.png" required />
      <button type="submit">Submit</button>
    </form>
  </div>
</body>
</html><?php
$conn = new mysqli("localhost", "root", "", "ashok_enterprise");
if ($conn->connect_error) {
  die("Connection failed: " . $conn->connect_error);
}

$name = $_POST['name'];
$phone = $_POST['phone'];
$email = $_POST['email'];
$uploadDir = "uploads/";
$fileName = time() . "_" . basename($_FILES["document"]["name"]);
$targetFile = $uploadDir . $fileName;

if (move_uploaded_file($_FILES["document"]["tmp_name"], $targetFile)) {
  $sql = "INSERT INTO form_data (name, phone, email, document_path) VALUES ('$name', '$phone', '$email', '$targetFile')";
  if ($conn->query($sql) === TRUE) {
    echo "✅ Form submitted successfully!";
  } else {
    echo "❌ Error: " . $conn->error;
  }
} else {
  echo "❌ Error uploading file.";
}
$conn->close();
?><!DOCTYPE html>
<html>
<head>
  <title>Admin Login</title>
</head>
<body>
  <form action="admin-check.php" method="POST">
    <h2>Admin Login</h2>
    <input type="text" name="username" placeholder="Username" required />
    <input type="password" name="password" placeholder="Password" required />
    <button type="submit">Login</button>
  </form>
</body>
</html><?php
session_start();
$user = $_POST['username'];
$pass = $_POST['password'];

if ($user == "admin" && $pass == "123456") {
  $_SESSION['admin'] = true;
  header("Location: admin.php");
} else {
  echo "<script>alert('Invalid login');window.location.href='admin-login.php';</script>";
}
?><?php
session_start();
if (!isset($_SESSION['admin'])) {
  header("Location: admin-login.php");
  exit();
}
$conn = new mysqli("localhost", "root", "", "ashok_enterprise");
$result = $conn->query("SELECT * FROM form_data ORDER BY submitted_at DESC");
?>

<!DOCTYPE html>
<html>
<head>
  <title>Admin Panel</title>
</head>
<body>
  <h2>Submitted Forms</h2>
  <a href="logout.php">Logout</a>
  <table border="1">
    <tr>
      <th>ID</th><th>Name</th><th>Phone</th><th>Email</th><th>Document</th><th>Time</th>
    </tr>
    <?php while($row = $result->fetch_assoc()) { ?>
    <tr>
      <td><?= $row['id'] ?></td>
      <td><?= $row['name'] ?></td>
      <td><?= $row['phone'] ?></td>
      <td><?= $row['email'] ?></td>
      <td><a href="<?= $row['document_path'] ?>" target="_blank">View</a></td>
      <td><?= $row['submitted_at'] ?></td>
    </tr>
    <?php } ?>
  </table>
</body>
</html><?php
session_start();
session_destroy();
header("Location: admin-login.php");
?>Options -IndexesCREATE DATABASE ashok_enterprise;
USE ashok_enterprise;

CREATE TABLE form_data (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  phone VARCHAR(15),
  email VARCHAR(100),
  document_path VARCHAR(255),
  submitted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
