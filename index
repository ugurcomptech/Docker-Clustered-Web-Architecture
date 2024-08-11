<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Container Bilgileri</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #e0f7fa;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            color: #333;
        }

        .container {
            text-align: center;
            background: #ffffff;
            border-radius: 12px;
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
            padding: 30px;
            max-width: 500px;
            width: 100%;
        }

        .logo {
            width: 120px;
            height: auto;
            margin-bottom: 20px;
        }

        h1 {
            margin: 0 0 20px;
            color: #00796b;
        }

        p {
            font-size: 1.2em;
            color: #555;
            margin: 10px 0;
        }

        strong {
            color: #00796b;
        }
    </style>
</head>
<body>
    <div class="container">
        <img src="https://i.pinimg.com/originals/5c/bb/a7/5cbba74b40ec0c0ce77b3db3ec1a5e05.png" alt="Docker Logo" class="logo">
        <?php
        // Hostname ve IP adresini al
        $hostname = gethostname();
        $ip_address = getenv('SERVER_ADDR'); 

        // Verileri ekrana yazdır
        echo "<h1>Container Bilgileri</h1>";
        echo "<p><strong>Hostname:</strong> $hostname</p>";
        echo "<p><strong>IP Adresi:</strong> $ip_address</p>";
        ?>
    </div>
</body>
</html>
