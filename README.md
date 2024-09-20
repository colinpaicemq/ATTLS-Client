# ATTLS-Sample 
A very basic TCPIP server sample on z/OS to illustrate the use of a managed AT-TLS connection
Files  are
 
1.   TCPBST   this attaches a thread to do all of the work
2.  CCTCPST to compile it  
3.  ATTLSST  the source of the thread, which does most of the work.  It uses BPX1SOC and BPX1OPT to set options (such as reuse)  
4.  ATTLS    to     query ATTLS and extract information about the connect, such as AT-TLS rule name
5. ATTLSPR  to print the information returned from ATTLS.   Eg the cipher  spec,TLS level
6. ATTLSGC  to get the certificate, and use pthread_security_applid_np to switch to the userid
7. NOERR    convert C error codes like ERANGE to string "Range error"
8. ATTLSSTA start the connection
9. ATTLSH   Header file to define data types
10. BASE64EN      base 64 encoding and decoding
11. PRINTHEX  to print a string in HEX with EBCDIC and ASCII
12. PAGENTTN my poicy agent files.   Start with COLATTLJ  (port 4000) 


You invoke it using

```
openssl s_client  -connect 10.1.1.2:4000  -CAfile doczosca.pem 
  -key  docec256.key.pem -certform PEM 
```
