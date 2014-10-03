---
title: "Sending unicode email headers in PHP"
date: 2009-08-17
---

I'm probably the last guy on earth to know this, but I'm recording it here just in case. I'm sending record titles in the subject line of emails, and of course they may be unicode. The body takes care of itself, but you need to explicitly encode a header like "Subject."

~~~PHP
    

    $headers['To'] = $to;
    $headers['From'] = $from;
    $headers['Content-Type'] = "text/plain; charset=utf-8";
    $headers['Content-Transfer-Encoding'] = "8bit";
    $b64subject = "=?UTF-8?B?" . base64_encode($subject) . "?=";
    $headers['Subject'] = $b64subject;

    $mail =& Mail::factory('sendmail', array('host' => $host,
                                             'port'=>$port));
    $retval =  $mail->send($to, $headers, $body);

~~~
