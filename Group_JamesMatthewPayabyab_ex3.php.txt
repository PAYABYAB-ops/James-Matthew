<?php
// Directory where notes will be stored
$notes_dir = 'notes';

// Create the directory if it doesn't exist
if (!file_exists($notes_dir)) {
    mkdir($notes_dir, 0777, true);
}

// Handle form submissions
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $notepad_name = trim($_POST['notepad_name']);
    $content = $_POST['content'];

    // Sanitize filename and create file path
    $filename = $notes_dir . '/' . preg_replace('/[^a-zA-Z0-9_\-]/', '_', $notepad_name) . '.txt';

    // Save content to the file
    if (!empty($notepad_name)) {
        file_put_contents($filename, $content);
    }

    // Redirect to avoid form resubmission issues and clear the text fields
    header('Location: ' . $_SERVER['PHP_SELF'] . '?saved=true');
    exit;
}

// Load existing content if a notepad is selected
$current_file = isset($_GET['file']) ? $_GET['file'] : '';
$content = '';
$notepad_name = '';
if ($current_file && file_exists($current_file)) {
    $content = file_get_contents($current_file);
    $notepad_name = basename($current_file, '.txt');
}

// Get list of all existing notepads
$notepads = glob($notes_dir . '/*.txt');
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Personal Notepad</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #f3f4f6;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        .container {
            background-color: #ffffff;
            padding: 30px;
            width: 800px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
        }

        h1 {
            color: #333;
            font-size: 24px;
            margin-bottom: 20px;
        }

        form {
            margin-bottom: 20px;
        }

        textarea {
            width: 100%;
            height: 250px;
            border: 1px solid #ccc;
            border-radius: 5px;
            padding: 10px;
            font-size: 14px;
            font-family: 'Courier New', Courier, monospace;
            margin-bottom: 15px;
            resize: vertical;
        }

        input[type="text"], input[type="submit"], button {
            padding: 10px;
            font-size: 14px;
            border-radius: 5px;
            border: 1px solid #ccc;
            margin-bottom: 10px;
            width: calc(100% - 22px);
        }

        input[type="submit"], button {
            background-color: #28a745;
            color: white;
            cursor: pointer;
            width: auto;
            padding: 10px 20px;
            border: none;
        }

        input[type="submit"]:hover, button:hover {
            background-color: #218838;
        }

        button {
            background-color: #ff5c5c;
        }

        button:hover {
            background-color: #e85050;
        }

        .notepad-list {
            margin-top: 20px;
        }

        .notepad-list h2 {
            font-size: 18px;
            color: #333;
        }

        .notepad-list ul {
            list-style: none;
            padding: 0;
        }

        .notepad-list ul li {
            padding: 5px 0;
        }

        .notepad-list ul li a {
            text-decoration: none;
            color: #007bff;
        }

        .notepad-list ul li a:hover {
            text-decoration: underline;
        }
    </style>
    <script>
        function clearFields() {
            document.getElementById('notepad_name').value = '';
            document.getElementById('content').value = '';
        }

        // Auto-clear fields if the notes were successfully saved
        window.onload = function () {
            const urlParams = new URLSearchParams(window.location.search);
            if (urlParams.get('saved') === 'true') {
                clearFields();
                window.history.replaceState({}, document.title, window.location.pathname);
            }
        };
    </script>
</head>
<body>
    <div class="container">
        <h1>My Notepad</h1>

        <form action="" method="post">
            <input type="text" id="notepad_name" name="notepad_name" placeholder="Enter notepad name" value="<?php echo htmlspecialchars($notepad_name); ?>" required>
            <textarea name="content" id="content" placeholder="Write your notes here..."><?php echo htmlspecialchars($content); ?></textarea>
            <div>
                <input type="submit" value="Save Notes">
                <button type="button" onclick="clearFields()">Clear</button>
            </div>
        </form>

        <div class="notepad-list">
            <h2>Available Notepads</h2>
            <ul>
                <?php foreach ($notepads as $notepad): ?>
                    <li>
                        <a href="?file=<?php echo urlencode($notepad); ?>">
                            <?php echo htmlspecialchars(basename($notepad, '.txt')); ?>
                        </a>
                    </li>
                <?php endforeach; ?>
            </ul>
        </div>
    </div>
</body>
</html>
