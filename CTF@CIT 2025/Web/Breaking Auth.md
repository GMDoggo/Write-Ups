# About
"Say my username."

**Note**: this challenge instance will reset every 15 minutes. If a challenge is not responsive, you might need to wait until the next quarter hour.

[http://23.179.17.40:58001/](http://23.179.17.40:58001/)

![](Images/Pasted%20image%2020250428110620.png)

# Solve

Testing SQLI gets the following
```
heisenberg' 


Fatal error: Uncaught mysqli_sql_exception: You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'test'' at line 1 in /var/www/html/index.php:23 Stack trace: #0 /var/www/html/index.php(23): mysqli->query('SELECT * FROM u...') #1 {main} thrown in /var/www/html/index.php on line 23
```

I wasn't in the mood for manual testing for SQL Injection, so I ended up getting a request for the login and sending it through SQLMap for doing the testing and dumping the DB.

![](Images/Pasted%20image%2020250426135315.png)