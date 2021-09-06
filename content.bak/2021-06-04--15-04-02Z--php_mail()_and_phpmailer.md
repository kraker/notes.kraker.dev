---
title: PHP mail() and PHPMailer
date: 2021-06-04 15:04
---

## Resources

PHP mail()
https://www.inmotionhosting.com/support/email/using-the-php-mail-function-to-send-emails/

PHPMailer
https://www.inmotionhosting.com/support/email/using-phpmailer-to-send-mail-through-php/

PHP Mail() and PHPMailer
https://www.hostinger.com/tutorials/send-emails-using-php-mail

Test PHP mail()
https://www.arclab.com/en/kb/php/how-to-test-and-fix-php-mail-function.html

Official Docs
https://www.php.net/manual/en/function.mail.php


## Testing

### PHP mail()

Simple test script
```
<?PHP
$sender = 'someone@somedomain.tld';
$recipient = 'you@yourdomain.tld';

$subject = "php mail test";
$message = "php test message";
$headers = 'From:' . $sender;

if (mail($recipient, $subject, $message, $headers))
{
    echo "Message accepted";
}
else
{
    echo "Error: Message not accepted";
}
?>
```

### PHPMailer

Simple test script
```
<?php

require "vendor/autoload.php";

$robo = 'robot@example.com';

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;


$developmentMode = true;
$mailer = new PHPMailer($developmentMode);

try {
    $mailer->SMTPDebug = 2;
    $mailer->isSMTP();

    if ($developmentMode) {
    $mailer->SMTPOptions = [
        'ssl'=> [
        'verify_peer' => false,
        'verify_peer_name' => false,
        'allow_self_signed' => true
        ]
    ];
    }


    $mailer->Host = 'mail.example.com';
    $mailer->SMTPAuth = true;
    $mailer->Username = 'robot@example.com';
    $mailer->Password = 'password';
    $mailer->SMTPSecure = 'tls';
    $mailer->Port = 587;

    $mailer->setFrom('robot@example.com', 'Name of sender');
    $mailer->addAddress('joe@example.com', 'Name of recipient');

    $mailer->isHTML(true);
    $mailer->Subject = 'PHPMailer Test';
    $mailer->Body = 'This is a <b>SAMPLE<b> email sent through <b>PHPMailer<b>';

    $mailer->send();
    $mailer->ClearAllRecipients();
    echo "MAIL HAS BEEN SENT SUCCESSFULLY";

} catch (Exception $e) {
    echo "EMAIL SENDING FAILED. INFO: " . $mailer->ErrorInfo;
}
```

