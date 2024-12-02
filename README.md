# ATTLS-Sample 
A very basic TCPIP server sample on z/OS to illustrate the use of a managed AT-TLS connection.

This displays the connection information, and displays the userid associated with the certificate (if any)
```
//COLCOMPI   JOB 1,MSGCLASS=H,COND=(4,LE) 
//* Job execute TCP server using ATTLS 
//* uses port 4000 
// SET LOADLIB=COLIN.LOAD2
//START1   EXEC PGM=TCPATTLS,REGION=0M, 
//   PARM='4000' 
//STDENV DD * 
GSK_TRACE=0xff 
/* 
//STEPLIB  DD DISP=SHR,DSN=&LOADLIB 
//SYSERR   DD SYSOUT=*,DCB=(LRECL=200) 
//SYSERROR DD SYSOUT=*,DCB=(LRECL=200) 
//SYSOUT   DD SYSOUT=*,DCB=(LRECL=200) 
//SYSPRINT DD SYSOUT=*,DCB=(LRECL=200) 
//SYSABEND DD SYSOUT=*,DCB=(LRECL=200) 
//CEEDUMP  DD SYSOUT=*,DCB=(LRECL=200) 
//ENV DD * 
/* 
```

To map a certificate to a userid I used the RACF commands
```
RACDCERT DELMAP(LABEL('IBMUSER1Label))ID(IBMUSER) 
RACDCERT MAP ID(IBMUSER)  - 
   WITHLABEL('IBMUSER1Label') - 
   SDNFILTER('CN=colinpaice.O=cpwebuser.C=GB') 
RACDCERT LISTMAP ID(IBMUSER) 
SETROPTS RACLIST(DIGTNMAP, DIGTCRIT) REFRESH 
```

## To execute the sample.
FTP the loadmodule ATTLS.XMIT.BIN to z/OS as bin, with dataset LRECL 80 RECFM FB BLKSIZE 3200.
Then use TSO RECEIVE INDSN(....)


Files  are
 
1.  TCPBST   this attaches a thread to do all of the work
2.  CCTCPST to compile it  
3.  ATTLSST  the source of the thread, which does most of the work.  It uses BPX1SOC and BPX1OPT to set options (such as reuse)  
4.  ATTLS    to     query ATTLS and extract information about the connect, such as AT-TLS rule name
5.  ATTLSPR  to print the information returned from ATTLS.   Eg the cipher  spec,TLS level
6.  ATTLSGC  to get the certificate, and use pthread_security_applid_np to switch to the userid
7.  NOERR    convert C error codes like ERANGE to string "Range error"
8.  ATTLSSTA start the connection
9.  ATTLSH   Header file to define data types
10. BASE64EN      base 64 encoding and decoding
11. PRINTHEX  to print a string in HEX with EBCDIC and ASCII
12. PAGENTTN my policy agent files.   Start with COLATTLJ  (port 4000) 


You invoke it (from Linux) using

```
openssl s_client  -connect 10.1.1.2:4000  -CAfile doczosca.pem 
  -key  docec256.key.pem -certform PEM 
```
or 
```
curl --verbose --cacert ./doczosca.pem --cert ./colinpaice.pem:password 
--key ./colinpaice.key.pem 
--tlsv1.2 --tls-max 1.2 
https://10.1.1.2:4000/rest_of_data_in_url
```


