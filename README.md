<?php
session_start(); // 세션 시작

// 데이터베이스 연결 설정
$host = 'localhost';
$dbname = 'shopping_db';
$username = 'root';
$password = '';

try {
    // 데이터베이스 연결
    $conn = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8", $username, $password);
    $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("데이터베이스 연결 실패: " . $e->getMessage());
}

// 폼 처리
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // 회원가입 처리
    if (isset($_POST['register'])) {
        $user_id = $_POST['id']; // 아이디
        $name = $_POST['name']; // 성명
        $password = $_POST['pwd']; // 비밀번호
        $password_confirm = $_POST['pw2']; // 비밀번호 확인
        $phone = $_POST['pl'] . '-' . $_POST['p2'] . '-' . $_POST['p3'];
        $gender = $_POST['sex'];
        $address = $_POST['addr'];
        $memo = $_POST['memo'];

        // 비밀번호 확인
        if ($password !== $password_confirm) {
            $error_message = "비밀번호와 비밀번호 확인이 일치하지 않습니다.";
        } else {
            $hashed_password = password_hash($password, PASSWORD_DEFAULT); // 비밀번호 해시화

            // 데이터 삽입
            $query = "INSERT INTO users (username, name, password, phone, gender, address, memo) 
                      VALUES (:username, :name, :password, :phone, :gender, :address, :memo)";
            $stmt = $conn->prepare($query);
            $stmt->bindParam(':username', $user_id);
            $stmt->bindParam(':name', $name);
            $stmt->bindParam(':password', $hashed_password);
            $stmt->bindParam(':phone', $phone);
            $stmt->bindParam(':gender', $gender);
            $stmt->bindParam(':address', $address);
            $stmt->bindParam(':memo', $memo);

            if ($stmt->execute()) {
                $success_message = "회원가입이 완료되었습니다. 로그인 해주세요!";
            } else {
                $error_message = "회원가입 중 오류가 발생했습니다.";
            }
        }
    }

    // 로그인 처리
    if (isset($_POST['login'])) {
        $input_username = $_POST['username'];
        $input_password = $_POST['password'];

        $query = "SELECT * FROM users WHERE username = :username";
        $stmt = $conn->prepare($query);
        $stmt->bindParam(':username', $input_username);
        $stmt->execute();
        $user = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($user && password_verify($input_password, $user['password'])) {
            $_SESSION['username'] = $user['username']; // 세션에 사용자 정보 저장
            header("Location: index.php?page=category"); // 카테고리 페이지로 이동
            exit;
        } else {
            $error_message = "아이디 또는 비밀번호가 잘못되었습니다.";
        }
    }
}

// 로그아웃 처리
if (isset($_GET['logout'])) {
    session_destroy();
    header("Location: index.php");
    exit;
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>회원가입 및 로그인</title>
</head>
<body>
<?php
// 알림 메시지 출력
if (isset($error_message)) echo "<p style='color:red;'>$error_message</p>";
if (isset($success_message)) echo "<p style='color:green;'>$success_message</p>";

// 페이지에 따라 다른 화면 표시
if (isset($_GET['page']) && $_GET['page'] === 'category') {
    // 카테고리 페이지
    if (!isset($_SESSION['username'])) {
        header("Location: index.php");
        exit;
    }
    $categories = ['전자제품', '의류', '식품', '도서', '스포츠'];
    ?>
    <h2>환영합니다, <?php echo htmlspecialchars($_SESSION['username']); ?>님!</h2>
    <h3>카테고리 목록</h3>
    <ul>
        <?php foreach ($categories as $category): ?>
            <li><?php echo $category; ?></li>
        <?php endforeach; ?>
    </ul>
    <a href="index.php?logout=true">로그아웃</a>
    <?php
} else {
    // 회원가입 및 로그인 폼 표시
    ?>
    <h2>회원가입</h2>
    <form method="post" action="index.php">
        <input type="text" name="id" placeholder="아이디" required><br>
        <input type="text" name="name" placeholder="성명" required><br>
        <input type="password" name="pwd" placeholder="비밀번호" required><br>
        <input type="password" name="pw2" placeholder="비밀번호 확인" required><br>
        핸드폰: 
        <select name="pl" required>
            <option value="010">010</option>
            <option value="011">011</option>
        </select>
        - <input type="text" name="p2" maxlength="4" size="4" required>
        - <input type="text" name="p3" maxlength="4" size="4" required><br>
        성별: <input type="radio" name="sex" value="남자" checked> 남
              <input type="radio" name="sex" value="여자"> 여<br>
        주소: <input type="text" name="addr" required><br>
        남기고 싶은 글:<br>
        <textarea name="memo" rows="4" cols="50"></textarea><br>
        <button type="submit" name="register">회원가입</button>
    </form>

    <h2>로그인</h2>
    <form method="post" action="index.php">
        <input type="text" name="username" placeholder="아이디" required><br>
        <input type="password" name="password" placeholder="비밀번호" required><br>
        <button type="submit" name="login">로그인</button>
    </form>
    <?php
}
?>
</body>
</html>

		
		
