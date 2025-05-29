I'll convert the provided PHP web application code into a Markdown format, organizing the content to clearly present each file with its code and maintaining the structure of the original files. The notes section will be included at the end. Each file's content will be wrapped in a single `<xaiArtifact>` tag with the appropriate metadata.

```markdown
# Web Application Cheat Sheet

Below is the documentation for a restaurant booking web application, including the source code for each file and notes on the database and functionality.

## booking.php

This file handles table booking functionality, requiring user authentication and form validation.

```php
<?php
session_start();
require_once 'includes/config.php';
require_once 'includes/functions.php';

if (!isset($_SESSION['user'])) {
    header('Location: login.php');
    exit;
}

$error = '';
$success = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $date = $_POST['date'] ?? '';
    $time = $_POST['time'] ?? '';
    $guests = intval($_POST['guests'] ?? 0);
    $phone = $_POST['phone'] ?? '';

    if (!$date || !$time || $guests < 1 || $guests > 10 || !validatePhone($phone)) {
        $error = 'Пожалуйста, заполните все поля корректно.';
    } else {
        $data = [
            'date' => $date,
            'time' => $time,
            'guests' => $guests,
            'phone' => $phone
        ];
        if (createBooking($data)) {
            $success = 'Бронирование успешно отправлено на рассмотрение.';
        } else {
            $error = 'Ошибка при создания бронирования.';
        }
    }
}
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Забронировать столик</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
<div class="container">
    <h1>Забронировать столик</h1>
    <?php if ($error): ?>
        <div class="error"><?= htmlspecialchars($error) ?></div>
    <?php elseif ($success): ?>
        <div class="success"><?= htmlspecialchars($success) ?></div>
    <?php endif; ?>
    <form method="post" action="booking.php">
        <label for="date">Дата:</label>
        <input type="date" id="date" name="date" required>
        <label for="time">Время:</label>
        <input type="time" id="time" name="time" required>
        <label for="guests">Количество гостей (1-10):</label>
        <input type="number" id="guests" name="guests" min="1" max="10" required>
        <label for="phone">Телефон (+7(XXX)-XXX-XX-XX):</label>
        <input type="text" id="phone" name="phone" pattern="\+7\(\d{3}\)-\d{3}-\d{2}-\d{2}" required>
        <button type="submit">Забронировать</button>
    </form>
    <p><a href="index.php">На главную</a></p>
</div>
</body>
</html>
```

## bookings.php

This file displays the user's bookings and personal information.

```php
<?php
session_start();
require_once 'includes/config.php';
require_once 'includes/functions.php';

if (!isset($_SESSION['user'])) {
    header('Location: login.php');
    exit;
}

$bookings = getUserBookings($_SESSION['user']['id']);
$user = $_SESSION['user'];
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Личный кабинет</title>
    <link rel="stylesheet" href="css/style.css">
    <style>
        table {border-collapse: collapse; width: 100%; max-width: 800px;}
        th, td {border: 1px solid #ddd; padding: 8px;}
        th {background: #f0f0f0;}
        .user-info {margin-bottom: 20px; max-width: 800px; border: 1px solid #ddd; padding: 15px; background: #fafafa; border-radius: 5px;}
        .user-info h2 {margin-top: 0;}
        .user-info p {margin: 5px 0;}
    </style>
</head>
<body>
<div class="container">
    <h1>Личный кабинет</h1>
    <div class="user-info">
        <h2>Личные данные</h2>
        <p><strong>Имя:</strong> <?= htmlspecialchars($user['first_name']) ?></p>
        <p><strong>Фамилия:</strong> <?= htmlspecialchars($user['last_name']) ?></p>
        <p><strong>Логин:</strong> <?= htmlspecialchars($user['username']) ?></p>
        <p><strong>Телефон:</strong> <?= htmlspecialchars($user['phone']) ?></p>
        <p><strong>Email:</strong> <?= htmlspecialchars($user['email']) ?></p>
    </div>
    <h2>Ваши бронирования</h2>
    <?php if (count($bookings) === 0): ?>
        <p>У вас пока нет бронирований.</p>
    <?php else: ?>
        <table>
            <tr>
                <th>Дата</th>
                <th>Время</th>
                <th>Гости</th>
                <th>Телефон</th>
                <th>Статус</th>
            </tr>
            <?php foreach ($bookings as $b): ?>
                <tr>
                    <td><?= htmlspecialchars($b['booking_date']) ?></td>
                    <td><?= htmlspecialchars($b['booking_time']) ?></td>
                    <td><?= (int)$b['guests'] ?></td>
                    <td><?= htmlspecialchars($b['phone']) ?></td>
                    <td><?= htmlspecialchars($b['status']) ?></td>
                </tr>
            <?php endforeach; ?>
        </table>
    <?php endif; ?>
    <p><a href="index.php">На главную</a></p>
</div>
</body>
</html>
```

## feedback.php

This file processes feedback submissions for bookings.

```php
<?php
require_once 'includes/config.php';
require_once 'includes/functions.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['booking_id'], $_POST['feedback'])) {
    $id = (int) $_POST['booking_id'];
    $feedback = trim($_POST['feedback']);
    addFeedback($id, $feedback);
}

header('Location: bookings.php');
exit;
?>
```

## index.php

The main page of the application, providing navigation based on user authentication status.

```php
<?php
session_start();
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Я буду кушац — Главная</title>
    <link rel="stylesheet" href="css/style.css">
    <style>
        body {font-family: sans-serif; background: #f4f4f4; margin: 0; padding: 0;}
        .container {max-width: 600px; margin: 80px auto; padding: 30px; background: white; box-shadow: 0 0 10px rgba(0,0,0,.1); text-align: center;}
        h1 {margin-bottom: 20px;}
        .btn {display: inline-block; padding: 10px 20px; margin: 10px; background: #007bff; color: white; text-decoration: none; border-radius: 4px;}
        .btn:hover {background: #0056b3;}
        .user-info {margin-top: 10px; font-size: 0.95em; color: #333;}
    </style>
</head>
<body>
<div class="container">
    <h1>Добро пожаловать в "Я буду кушац"</h1>
    <?php if (isset($_SESSION['user'])): ?>
        <p class="user-info">Вы вошли как: <strong><?= htmlspecialchars($_SESSION['user']['first_name'] . ' ' . $_SESSION['user']['last_name']) ?></strong></p>
        <?php if ($_SESSION['user']['role'] === 'admin'): ?>
            <a class="btn" href="admin/index.php">Панель администратора</a>
        <?php else: ?>
            <a class="btn" href="booking.php">Забронировать столик</a>
            <a class="btn" href="bookings.php">Личный кабинет</a>
        <?php endif; ?>
        <a class="btn" href="logout.php">Выйти</a>
    <?php else: ?>
        <a class="btn" href="login.php">Войти</a>
        <a class="btn" href="register.php">Зарегистрироваться</a>
    <?php endif; ?>
</div>
</body>
</html>
```

## login.php

Handles user login with form validation and redirects based on user role.

```php
<?php
session_start();
require_once 'includes/config.php';
require_once 'includes/functions.php';

$error = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username']);
    $password = trim($_POST['password']);

    if (empty($username) || empty($password)) {
        $error = 'Пожалуйста, введите логин и пароль.';
    } else {
        if (loginUser($username, $password)) {
            if ($_SESSION['user']['role'] === 'admin') {
                header('Location: admin/index.php');
            } else {
                header('Location: bookings.php');
            }
            exit;
        } else {
            $error = 'Неверный логин или пароль.';
        }
    }
}
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Вход | Я буду кушац</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
<div class="container">
    <h1>Вход</h1>
    <?php if (!empty($error)): ?>
        <div class="error"><?= htmlspecialchars($error) ?></div>
    <?php endif; ?>
    <form method="post" action="login.php">
        <label for="username">Логин:</label>
        <input type="text" id="username" name="username" placeholder="Введите логин" required>
        <label for="password">Пароль:</label>
        <input type="password" id="password" name="password" placeholder="Введите пароль" required>
        <button type="submit">Войти</button>
    </form>
    <p>Нет аккаунта? <a href="register.php">Зарегистрироваться</a></p>
    <p><a href="index.php"><button type="button">Главная</button></a></p>
</div>
</body>
</html>
```

## logout.php

Terminates the user session and redirects to the login page.

```php
<?php
session_start();
session_destroy();
header('Location: login.php');
exit;
?>
```

## register.php

Handles user registration with input validation and phone number formatting.

```php
<?php
session_start();
require_once 'includes/config.php';
require_once 'includes/functions.php';

$error = '';
$success = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $data = [
        'username'   => trim($_POST['username']),
        'password'   => $_POST['password'],
        'first_name' => trim($_POST['first_name']),
        'last_name'  => trim($_POST['last_name']),
        'phone'      => trim($_POST['phone']),
        'email'      => trim($_POST['email'])
    ];

    if (!validateUsername($data['username'])) {
        $error = 'Логин должен быть на кириллице и не менее 6 символов.';
    } elseif (!validatePassword($data['password'])) {
        $error = 'Пароль должен содержать не менее 6 символов.';
    } elseif (!validatePhone($data['phone'])) {
        $error = 'Неверный формат телефона. Используйте +7(XXX)-XXX-XX-XX.';
    } elseif (!validateEmail($data['email'])) {
        $error = 'Неверный формат электронной почты.';
    } else {
        $result = registerUser($data);
        if ($result === true) {
            $success = 'Регистрация прошла успешно! Перенаправление...';
            header("refresh:2;url=login.php");
        } elseif ($result === 'duplicate') {
            $error = 'Пользователь с таким логином уже существует.';
        } else {
            $error = 'Ошибка при регистрации. Попробуйте позже.';
        }
    }
}
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Регистрация</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
<div class="container">
    <h1>Регистрация</h1>
    <?php if ($error): ?>
        <div class="error"><?= htmlspecialchars($error) ?></div>
    <?php elseif ($success): ?>
        <div class="success"><?= htmlspecialchars($success) ?></div>
    <?php endif; ?>
    <form method="post" action="register.php">
        <label for="username">Логин (кириллица, ≥6 символов):</label>
        <input type="text" id="username" name="username" required>
        <label for="/password">Пароль (≥6 символов):</label>
        <input type="password" id="password" name="password" required>
        <label for="first_name">Имя:</label>
        <input type="text" id="first_name" name="first_name" required>
        <label for="last_name">Фамилия:</label>
        <input type="text" id="last_name" name="last_name" required>
        <label for="phone">Телефон (+7(XXX)-XXX-XX-XX):</label>
        <input type="text" id="phone" name="phone" required>
        <label for="email">Электронная почта:</label>
        <input type="email" id="email" name="email" required>
        <button type="submit">Зарегистрироваться</button>
    </form>
    <p>Уже есть аккаунт? <a href="login.php">Войти</a></p>
</div>
<script>
document.addEventListener('DOMContentLoaded', function() {
    const phoneInput = document.getElementById('phone');
    phoneInput.addEventListener('input', function(e) {
        let x = phoneInput.value.replace(/\D/g, '').substring(0, 11);
        let formatted = '+7(';
        if (x.length > 1) {
            formatted += x.substring(1, 4);
        }
        if (x.length >= 4) {
            formatted += ')-' + x.substring(4, 7);
        }
        if (x.length >= 7) {
            formatted += '-' + x.substring(7, 9);
        }
        if (x.length >= 9) {
            formatted += '-' + x.substring(9, 11);
        }
        phoneInput.value = formatted;
    });
    phoneInput.addEventListener('focus', function() {
        if (!phoneInput.value) {
            phoneInput.value = '+7(';
        }
    });
    phoneInput.addEventListener('blur', function() {
        if (phoneInput.value === '+7(') {
            phoneInput.value = '';
        }
    });
});
</script>
</body>
</html>
```

## includes/config.php

Configuration file for database connection.

```php
<?php
session_start();
$host = 'localhost';
$user = 'root';
$pass = '';
$db = 'restaurant_booking';

$conn = new mysqli($host, $user, $pass, $db);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
$conn->set_charset("utf8mb4");
?>
```

## includes/functions.php

Contains utility functions for validation, user management, and booking operations.

```php
<?php
function validatePhone($phone) {
    return preg_match('/^\+7\(\d{3}\)-\d{3}-\d{2}-\d{2}$/', $phone);
}

function validateEmail($email) {
    return filter_var($email, FILTER_VALIDATE_EMAIL);
}

function validateUsername($username) {
    return preg_match('/^[а-яА-ЯёЁ]{6,}$/u', $username);
}

function validatePassword($password) {
    return strlen($password) >= 6;
}

function registerUser($data) {
    global $conn;
    $stmt = $conn->prepare("SELECT id FROM users WHERE username = ?");
    $stmt->bind_param("s", $data['username']);
    $stmt->execute();
    $stmt->store_result();
    if ($stmt->num_rows > 0) {
        return 'duplicate';
    }
    $stmt = $conn->prepare("INSERT INTO users (username, password, first_name, last_name, phone, email, role) 
                            VALUES (?, ?, ?, ?, ?, ?, 'user')");
    $stmt->bind_param("ssssss", 
        $data['username'],
        $data['password'],
        $data['first_name'],
        $data['last_name'],
        $data['phone'],
        $data['email']
    );
    if ($stmt->execute()) {
        return true;
    } else {
        return 'error';
    }
}

function loginUser($username, $password) {
    global $conn;
    $stmt = $conn->prepare("SELECT * FROM users WHERE username = ?");
    $stmt->bind_param("s", $username);
    $stmt->execute();
    $result = $stmt->get_result();
    if ($user = $result->fetch_assoc()) {
        if ($password === $user['password']) {
            $_SESSION['user'] = $user;
            return true;
        }
    }
    return false;
}

function addFeedback($booking_id, $feedback) {
    global $conn;
    $stmt = $conn->prepare("INSERT INTO feedback (booking_id, feedback) VALUES (?, ?)");
    $stmt->bind_param("is", $booking_id, $feedback);
    return $stmt->execute();
}

function createBooking($data) {
    global $conn;
    $user_id = $_SESSION['user']['id'];
    $stmt = $conn->prepare("INSERT INTO bookings (user_id, booking_date, booking_time, guests, phone, status) VALUES (?, ?, ?, ?, ?, 'new')");
    $stmt->bind_param("issis", $user_id, $data['date'], $data['time'], $data['guests'], $data['phone']);
    return $stmt->execute();
}

function getUserBookings($user_id) {
    global $conn;
    $stmt = $conn->prepare("SELECT * FROM bookings WHERE user_id = ? ORDER BY booking_date DESC");
    $stmt->bind_param("i", $user_id);
    $stmt->execute();
    return $stmt->get_result()->fetch_all(MYSQLI_ASSOC);
}

function getAllBookings() {
    global $conn;
    $sql = "SELECT b.id, b.booking_date, b.booking_time, b.guests, b.phone, b.status, u.first_name, u.last_name FROM bookings b JOIN users u ON b.user_id = u.id ORDER BY b.booking_date DESC, b.booking_time DESC";
    $res = $conn->query($sql);
    return $res->fetch_all(MYSQLI_ASSOC);
}

function updateBookingStatus($id, $status) {
    global $conn;
    $stmt = $conn->prepare("UPDATE bookings SET status = ? WHERE id = ?");
    $stmt->bind_param("si", $status, $id);
    return $stmt->execute();
}
?>
```

## css/style.css

Stylesheet for the application's user interface.

```css
body {
    font-family: Arial, sans-serif;
    background-color: #f2f2f2;
}

.container {
    width: 600px;
    margin: auto;
    background: #fff;
    padding: 20px;
    margin-top: 40px;
    box-shadow: 0 0 10px rgba(0,0,0,0.1);
}

h1, h2 {
    text-align: center;
}

input, textarea, select, button {
    width: 100%;
    padding: 10px;
    margin: 10px 0;
    box-sizing: border-box;
}

.error {
    color: red;
    margin-bottom: 10px;
}

a {
    display: inline-block;
    margin-top: 10px;
    color: blue;
    text-decoration: none;
}
```

## admin/index.php

Admin panel for managing booking statuses.

```php
<?php
session_start();
require_once '../includes/config.php';
require_once '../includes/functions.php';

if (!isset($_SESSION['user']) || $_SESSION['user']['role'] !== 'admin') {
    header('Location: ../login.php');
    exit;
}

$error = '';
$success = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['booking_id'], $_POST['status'])) {
    $booking_id = intval($_POST['booking_id']);
    $status = $_POST['status'];
    $allowed_statuses = ['new', 'completed', 'cancelled'];
    if (in_array($status, $allowed_statuses)) {
        if (updateBookingStatus($booking_id, $status)) {
            $success = 'Статус бронирования обновлен.';
        } else {
            $error = 'Ошибка при обновлении статуса.';
        }
    } else {
        $error = 'Недопустимый статус.';
    }
}

$bookings = getAllBookings();
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Панель администратора</title>
    <link rel="stylesheet" href="../css/style.css">
    <style>
        table {border-collapse: collapse; width: 100%; max-width: 900px;}
        th, td {border: 1px solid #ddd; padding: 8px;}
        th {background: #f0f0f0;}
        select, button {padding: 5px;}
    </style>
</head>
<body>
<div class="container">
    <h1>Панель администратора</h1>
    <?php if ($error): ?>
        <div class="error"><?= htmlspecialchars($error) ?></div>
    <?php elseif ($success): ?>
        <div class="success"><?= htmlspecialchars($success) ?></div>
    <?php endif; ?>
    <?php if (count($bookings) === 0): ?>
        <p>Нет бронирований.</p>
    <?php else: ?>
        <table>
            <tr>
                <th>Пользователь</th>
                <th>Дата</th>
                <th>Время</th>
                <th>Гости</th>
                <th>Телефон</th>
                <th>Статус</th>
                <th>Обновить статус</th>
            </tr>
            <?php foreach ($bookings as $b): ?>
                <tr>
                    <td><?= htmlspecialchars($b['first_name'] . ' ' . $b['last_name']) ?></td>
                    <td><?= htmlspecialchars($b['booking_date']) ?></td>
                    <td><?= htmlspecialchars($b['booking_time']) ?></td>
                    <td><?= (int)$b['guests'] ?></td>
                    <td><?= htmlspecialchars($b['phone']) ?></td>
                    <td><?= htmlspecialchars($b['status']) ?></td>
                    <td>
                        <form method="post" action="index.php">
                            <input type="hidden" name="booking_id" value="<?= (int)$b['id'] ?>">
                            <select name="status">
                                <option value="new" <?= $b['status'] === 'new' ? 'selected' : '' ?>>Новое</option>
                                <option value="completed" <?= $b['status'] === 'completed' ? 'selected' : '' ?>>Посещение состоялось</option>
                                <option value="cancelled" <?= $b['status'] === 'cancelled' ? 'selected' : '' ?>>Отменено</option>
                            </select>
                            <button type="submit">Обновить</button>
                        </form>
                    </td>
                </tr>
            <?php endforeach; ?>
        </table>
    <?php endif; ?>
    <p><a href="../index.php">На главную</a> | <a href="../logout.php">Выйти</a></p>
</div>
</body>
</html>
```

## Notes

- **Database**: Uses MySQL (`restaurant_booking`).
- **Structure**: PHP for server-side logic, HTML/CSS for interface, JavaScript for phone number formatting.
- **Validation**: 
  - Phone: Format `+7(XXX)-XXX-XX-XX`.
  - Username: Cyrillic, ≥6 characters.
  - Password: ≥6 characters.
  - Email: Standard email format.
- **Sessions**: Authentication checked at the start of each protected file.
```
