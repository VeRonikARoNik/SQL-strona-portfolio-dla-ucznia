Portfolio projektów (PHP + MySQL)

1. OPIS PROJEKTU
----------------
Projekt jest aplikacją internetową typu portfolio, wykonaną w PHP
z bazą danych MySQL. Aplikacja pozwala dodawać oraz wyświetlać projekty.

Każdy projekt zawiera:
- tytuł
- opis
- technologie
- link do demo (opcjonalnie)
- obraz (upload)
- datę dodania

Użytkownik dodaje projekt przez formularz na stronie, dane zapisywane są
w bazie danych, a zdjęcie projektu w katalogu "uploads/".


2. TECHNOLOGIE
--------------
- PHP 7/8
- MySQL / MariaDB
- HTML5
- CSS3
- Apache (XAMPP)
- phpMyAdmin


3. WYMAGANIA
------------
- serwer Apache + MySQL (np. XAMPP)
- przeglądarka internetowa
- phpMyAdmin do obsługi bazy danych


4. STRUKTURA KATALOGÓW
----------------------

portfolio_zajecia/
├─ css/
│  └─ style.css      <-- plik stylów CSS
├─ uploads/          <-- katalog na dodawane obrazy
├─ config.php        <-- konfiguracja połączenia z bazą
└─ index.php         <-- główna strona aplikacji


5. BAZA DANYCH (SQL)
--------------------

-- utworzenie bazy danych

```
CREATE DATABASE portfolio_zajecia
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

USE portfolio_zajecia;
```
-- utworzenie tabeli projektów

```
CREATE TABLE projekty (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    tytul VARCHAR(150) NOT NULL,
    opis TEXT NOT NULL,
    technologie VARCHAR(255) DEFAULT NULL,
    link_demo VARCHAR(255) DEFAULT NULL,
    obrazek VARCHAR(255) DEFAULT NULL,
    data_dodania TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

6. PRZYKŁADOWE DANE W BAZIE
---------------------------

Poniżej przykładowe wypełnienie tabeli "projekty" (7 projektów).
Obrazek pozostaje pusty (NULL), obrazy będą dodane później przez formularz.
```
INSERT INTO projekty (tytul, opis, technologie, link_demo)
VALUES
('Strona wizytówka',
 'Prosta strona przedstawiająca osobę, dane kontaktowe i sekcję o mnie.',
 'HTML, CSS',
 NULL),

('Sklep internetowy mini',
 'Prototyp sklepu internetowego z listą produktów i koszykiem.',
 'PHP, MySQL, HTML, CSS',
 NULL),

('Blog PHP',
 'Aplikacja blogowa umożliwiająca dodawanie i wyświetlanie wpisów.',
 'PHP, MySQL',
 NULL),

('Kalkulator BMI',
 'Strona obliczająca wskaźnik masy ciała na podstawie wzrostu i wagi.',
 'JavaScript, HTML, CSS',
 NULL),

('Katalog filmów',
 'Lista filmów z opisami, ocenami i podstawowymi informacjami.',
 'PHP, MySQL, HTML',
 NULL),

('To-do lista',
 'Aplikacja do zarządzania zadaniami (dodawanie, lista zadań do wykonania).',
 'PHP, MySQL, CSS',
 NULL),

('Panel logowania',
 'Prosty panel logowania użytkowników z weryfikacją hasła.',
 'PHP, MySQL, Sessions',
 NULL);

```
7. KONFIGURACJA POŁĄCZENIA (config.php)
---------------------------------------

Plik: config.php
```
<?php
$db_host = 'localhost';
$db_user = 'root';
$db_pass = '';
$db_name = 'portfolio_zajecia';

$mysqli = new mysqli($db_host, $db_user, $db_pass, $db_name);

if ($mysqli->connect_error) {
    die('Błąd połączenia z bazą danych: ' . $mysqli->connect_error);
}

$mysqli->set_charset('utf8mb4');
?>
```

8. KOD STRONY (index.php)
-------------------------

Plik: index.php
```
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

require_once 'config.php';

$blad = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $tytul = trim($_POST['tytul'] ?? '');
    $opis = trim($_POST['opis'] ?? '');
    $technologie = trim($_POST['technologie'] ?? '');
    $link_demo = trim($_POST['link_demo'] ?? '');
    $obrazek = null;

    // Sprawdzenie wymaganych pól
    if ($tytul === '' || $opis === '') {
        $blad = 'Tytuł i opis są obowiązkowe.';
    } else {
        // Obsługa uploadu pliku (opcjonalnie)
        if (!empty($_FILES['obrazek']['name'])) {
            $nazwa = $_FILES['obrazek']['name'];
            $tmp = $_FILES['obrazek']['tmp_name'];
            $rozszerzenie = strtolower(pathinfo($nazwa, PATHINFO_EXTENSION));
            $dozwolone = ['jpg', 'jpeg', 'png'];

            if (in_array($rozszerzenie, $dozwolone)) {
                $nowa_nazwa = time() . '_' . rand(1000, 9999) . '.' . $rozszerzenie;
                $sciezka = 'uploads/' . $nowa_nazwa;

                if (move_uploaded_file($tmp, $sciezka)) {
                    $obrazek = $sciezka;
                }
            }
        }

        // Zapis projektu do bazy danych
        $stmt = $mysqli->prepare("
            INSERT INTO projekty (tytul, opis, technologie, link_demo, obrazek)
            VALUES (?, ?, ?, ?, ?)
        ");

        if ($stmt) {
            $stmt->bind_param('sssss', $tytul, $opis, $technologie, $link_demo, $obrazek);
            $stmt->execute();
            $stmt->close();
        } else {
            $blad = 'Błąd przygotowania zapytania.';
        }
    }
}

// Pobranie projektów z bazy
$projekty = [];
$result = $mysqli->query("SELECT * FROM projekty ORDER BY data_dodania DESC");
if ($result) {
    while ($row = $result->fetch_assoc()) {
        $projekty[] = $row;
    }
    $result->free();
}
?>

<!DOCTYPE html>
<html lang="pl">
<head>
<meta charset="UTF-8">
<title>Portfolio projektów</title>
<link rel="stylesheet" href="css/style.css">
</head>
<body>

<h1>Portfolio projektów</h1>

<?php if ($blad): ?>
<p style="color:red;"><?php echo htmlspecialchars($blad, ENT_QUOTES, 'UTF-8'); ?></p>
<?php endif; ?>

<form method="post" enctype="multipart/form-data">
    <p>
        <input type="text" name="tytul" placeholder="Tytuł projektu" required>
    </p>
    <p>
        <textarea name="opis" placeholder="Opis projektu" required></textarea>
    </p>
    <p>
        <input type="text" name="technologie" placeholder="Technologie (np. PHP, HTML, CSS)">
    </p>
    <p>
        <input type="url" name="link_demo" placeholder="https://example.com">
    </p>
    <p>
        <input type="file" name="obrazek" accept="image/*">
    </p>
    <p>
        <button type="submit">Dodaj projekt</button>
    </p>
</form>

<h2>Lista projektów</h2>

<?php if (empty($projekty)): ?>
    <p>Brak projektów w bazie.</p>
<?php else: ?>
    <?php foreach ($projekty as $p): ?>
        <div style="border:1px solid #ccc; padding:10px; margin-bottom:10px;">
            <h3><?php echo htmlspecialchars($p['tytul'], ENT_QUOTES, 'UTF-8'); ?></h3>

            <?php if (!empty($p['obrazek'])): ?>
                <img src="<?php echo htmlspecialchars($p['obrazek'], ENT_QUOTES, 'UTF-8'); ?>"
                     alt="Obraz projektu"
                     style="max-width:200px; display:block; margin-bottom:8px;">
            <?php endif; ?>

            <p><?php echo nl2br(htmlspecialchars($p['opis'], ENT_QUOTES, 'UTF-8')); ?></p>

            <?php if (!empty($p['technologie'])): ?>
                <p><strong>Technologie:</strong> <?php echo htmlspecialchars($p['technologie'], ENT_QUOTES, 'UTF-8'); ?></p>
            <?php endif; ?>

            <?php if (!empty($p['link_demo'])): ?>
                <p><a href="<?php echo htmlspecialchars($p['link_demo'], ENT_QUOTES, 'UTF-8'); ?>" target="_blank">
                    Zobacz demo
                </a></p>
            <?php endif; ?>

            <p><small>Dodano: <?php echo htmlspecialchars($p['data_dodania'], ENT_QUOTES, 'UTF-8'); ?></small></p>
        </div>
    <?php endforeach; ?>
<?php endif; ?>

</body>
</html>
```

9. STYLOWANIE STRONY (css/style.css)
------------------------------------

Plik: css/style.css
```
/* Prosty styl strony – uczeń może go rozbudować */

* {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f3f4f6;   /* TUTAJ UCZEŃ MA ZMIENIĆ TŁO W ZADANIU */
    color: #111827;
}

h1 {
    text-align: center;
    padding: 20px;
    background: #111827;
    color: #f9fafb;
    margin: 0 0 20px 0;
}

form {
    max-width: 600px;
    margin: 0 auto 20px;
    padding: 10px;
    background: #ffffff;
    border-radius: 8px;
    box-shadow: 0 2px 6px rgba(0,0,0,0.05);
}

form p {
    margin: 10px 0;
}

input[type="text"],
input[type="url"],
textarea,
input[type="file"] {
    width: 100%;
    padding: 8px;
    font: inherit;
}

button {
    padding: 8px 14px;
    font: inherit;
    cursor: pointer;
}

h2 {
    max-width: 800px;
    margin: 0 auto 10px;
}

div {
    max-width: 800px;
    margin: 0 auto;
}
```

10. INSTRUKCJA URUCHOMIENIA
---------------------------

1. Zainstalować i uruchomić XAMPP (Apache + MySQL).
2. Wejść na http://localhost/phpmyadmin.
3. Wykonać kod SQL z punktu 5, a następnie wstawić przykładowe dane z punktu 6.
4. W katalogu htdocs utworzyć folder "portfolio_zajecia".
5. Skopiować pliki: index.php, config.php, katalog "css" oraz katalog "uploads".
6. Utworzyć pusty folder "uploads" (do przechowywania obrazów).
7. Uruchomić stronę w przeglądarce:
   http://localhost/portfolio_zajecia/


11. ZADANIA DO WYKONANIA 
--------------------------------

1) W pliku css/style.css ZMIENIĆ tło strony (właściwość background w body)
   na inny kolor lub np. gradient. Zmiana ma być widoczna po odświeżeniu strony.

2) DODAĆ SAMODZIELNIE 3 PROJEKTY:
   - poprzez formularz na stronie,
   - każdy projekt ma mieć tytuł i opis,
   - do każdego projektu dodać obraz (plik pobrany z internetu),
   - obrazy mają zostać zapisane w folderze "uploads/" przez mechanizm uploadu.


12. CEL EDUKACYJNY
------------------
Projekt służy do nauki:
- tworzenia formularzy w HTML,
- obsługi danych w PHP,
- komunikacji z bazą MySQL (INSERT, SELECT),
- uploadu plików na serwer,
- prostego stylowania strony w CSS.



