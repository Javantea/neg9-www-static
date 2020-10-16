---
title: "OpenCTF 2015 - signfasterer (crypto 400) Writeup"
slug: "openctf-2015-signfasterer-crypto-400-writeup"
date: "2015-08-12 09:27:59.701317"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

Hint:

> We have new signature requirements from Central Licensing.
> 
> We're happy to comply, but,...it's a bit slow. Can you figure it out?
> 
> Service: 10.0.66.121:8008 Reference Client: [172.16.18.20/sign_faster_client-d57fe60ab04625921121656d3695beca](https://neg9.org/resources/media/openctf-2015-signfastest-crypto-100-writeup/sign_faster_client-d57fe60ab04625921121656d3695beca)

The library four1c2.py is the central part of the solution. The reason for foul naming of the class is because using this in any system is a horrible weakness and should never be used for any reason, especially not performance. fasterer.py and four1c2.py both use an unnecessary library gnfs1.py which I am not releasing due to code quality. It provides a list of possible primes using a sieve.

The library four1c2.py provides fuckedRSA, a class that is able to compute Chinese Remainder Theorem quickly with an arbitrary number of primes. A discussion of how CRT works is below (same as the solution from signfaster).

Say you have the primes [3, 5, 7, 11, 13, 15] as your prime factors. N is 3*5*7*11*13*15 = 225225.
The decryption exponent d is computed from the encryption exponent e and phi which is a combination of the prime factors.
phi = (p-1)*(q-1)*...
In the example phi = 2*4*6*10*12*14 = 80640
Let's say that e = 31
d = inverse(e, phi)
In this case, d would be 62431, a large number compared to N.
Encryption works like this:
ciphertext = pow(plaintext, e, N)
Let's say our ciphertext is 110138.
Then the decryption occurs like:
plaintext = pow(ciphertext, d, N)
In this case plaintext = 1337.
In math notation:
plaintext = (ciphertext ^ d) mod N
This is an expensive method of decrypting (or signing which is the same operation in RSA) a piece of data.
The Chinese Remainder Theorem states that you can compute the decryption with much lower exponents (and thus faster) by splitting the decryption into multiple parts and combining.
Compute:
q_inv = Crypto.Util.number.inverse(q, p)
m_1 = pow(ct, d_P, p)
m_2 = pow(ct, d_Q, q)
h = (q_inv*(m_1-m_2)) % p
m = m_2 + h * q

The d_P for each factor can be computed as d mod (p-1) where p is factor.
The set of d_P for our example would be [1, 3, 1, 1, 7, 5].
q_inv is the inverse of q (the second prime) mod p.
The first q_inv is inverse(3, 5) = 2.

The solution required a lot of iteration and I was concerned that it wouldn't solve it before the competition was over. A failed attempt to improve speed was brute force of e to get a lower d value, but that saves maybe 10% time. The major improvement over the signfaster solution was using gmpy2. gmpy2 is perhaps 50 or 100 times faster than python's pow function or the Crypto.PublicKey.RSA.sign function. If you're ever using those, you should know how slow they are.

The winning solution was this:

{language=python}
~~~~~~~~
python fasterer.py 10.0.66.121 8010
Connected
lineReceived preauth    Please burn some cpu cycles first: g7yfH/YYaVNWS7DT 8589934
Doing proof of work...
sendLine     pubkey     1439074140000000500
lineReceived pubkey     Public key?
Generating RSA key...
('e=', mpz(2360862523), 32)
sendLine     challange  MIICJTANBgkqhkiG9w0BAQEFAAOCAhIAMIICDQKCAgIiJYmF9Ug2kdau58zXCfBwQV4rxocanVn5RAGDvdMAjCPBXqOn0RuOgy0Xz6ERWUZ0wze1gY9Ll4GI/EzA/lOtOmLQd7wYll5yLoXFgTHKEoQY0Nsd35h4SXL99HctC/q/e3CLWhuVi4BsABAS4T4FedE2kIxHJwbhBxVbjXWnAYph0Y1R5p4fXx2IxHCmL5jnAG48iKez2gcCEMkaKY+Edsx7XTcRvRz5blpDWoRMxGdEvwn9UZH3rcKFzNKooqhfACX+1Ljre8uZjcML7b7eGjSlMigPxVr4rYZLSZi4BAbW8yvvZpgNvRoAmqEFJ3WdWb0ii0hNJ0BgAZ8bAzpho1XB776En+rUeCEr6TAFJWk0tj+eo7zOTUHeK7/l9QMa+nOvkjm+eA9pmLGHBMC2xgvcnxNopJBym6x2PqQ1djbge9Sn579hsxHLg3/N+ZBuDGZa5NgtiqIuuHD2NbyAMI+IveEgGmnm7DspM/z6DDTpjHNR8lHjeceA+9sd6jMu1YE6F/+faprPYb20nlRJy6NNpNcng9ChaDkzyV7BJ6KCe/02etP6sEmPI6MuZTGZmoM4RUJxw20wkodYmWlwE89XLg/8R6Vpr5Km04JV2uJYYEX/nvSq4beSJVR1uS/vPuDov5O4sCORmOYyF20nPL36iRbeJU0hqz7ZvnuEofkpAgUAjLfnOw==
lineReceived challange  Okay, make it quick, your challange is: 0AryFTCqDPooIyZrhhYqS5CNd0RmBA59jOxYOzZPcCc=
('do challenge!', '0AryFTCqDPooIyZrhhYqS5CNd0RmBA59jOxYOzZPcCc=')
sssssssssssss 250006012682151406846537376154540687961367589174709393533010073118257746557269158620641883605800137347701463346733284687994828814330266067691227064431317695890024521694239859003004921843870683222275127225088083581413471691965400893967067431316070190479281055324479854889284736372639887191032476462066284435557767235922928197165385809810895845459602715414945988666201380436927316134276486670353961513184015608213645383345217075032195552353704236282426066569390996656781785865466639058709936463884750521324897094499503044764825649854577365168000777613841561265621681005976320694977151156410704776528773526285069363400900280435873306410416520339826247717001078396600837779678027751967330792562732188473088526742379324885571851707000835556319076139465054535608837650383524555443200802064809030540548538769656814909186435996477495136599864493969163471781349581266462475218997858128280406681531279353475298670890480893236349909903781531853969764798675037315631735105549632889840362493478791827508451264649077861585927406934858265070349001578926922971409241379327569742100924765595093006804285197262633426183358425598804761352917119694354193766328874694602285078109206054552725656917812615652033037247763398207046663866135697718627262182438406219057601852594695222737034247272029639328224997744558255202897
x 0.788444042206
sendLine     print      AkXmqQT7JPYNgAEzF/mT/zP0n6BwMTnu6fSsljbVNu671DA94WiQMqNGpv74l3tZohm22m6/jnfeAHBQRR+96LZtkcm1EoZb9jdC9FdyD9i7eq93/jwjxVj5J9HgFNbWR7axZn1zDnia8bJRZNoA3/7v7u+oLH/lcJg8iwjmnz6YOzFS7pSlAdO2kXMb3CY6PJChvPMUTTPrPVhyPls28j86cVWEk4uCrf5q+BUz4qCfzgsJJdgnjTf+Fh8wA1ooBArAlf9PYUuVG6uj6jPZ5MwM2LRQubvxXWDj3QqD5SE7fggJQIMJuRsN1Yvw1vplaNx3RcNLEDRogkuebRtFHsW7aiSrOWaPfYdbpz0vMX/mwv9wISFvp2xR453Ndn6MRyWLOCttfxgKuAlbKR1eIrMswChzFJPosJlXPFaQIxqkYu4uFn4tBsFksXPMPx6/4LHfwiV+3weOs2fNMSnKrrRU/f8RoVTOVAVORW/hRIog6rULSpfsgqTTt6a3l0HUckhDKi+anBlIlHRCO4ENF+U5I91NmPQTsQBXubwtYS3oihV8GKRpRvktBSYdlBb7U5JjIH27fC15RknZK3qcnTvTgxgvVfRtAmo7S32SWjVfl4Or8yqiVnvYMsD7+i3+jWK5ZtO/QoLoL6PGL48//RJO31v7zVjOYrOBV886nAOM4iuNYvfSoRuiEXgdJ1Ko/vhJn7NqIFCO4WJR
lineReceived print      Solution verified sucessfully!
This is what they gave us: 'Solution verified sucessfully!'
lineReceived print      You took 0.790624 seconds.
This is what they gave us: 'You took 0.790624 seconds.'
lineReceived print      400 points (flag goes in signfasterer). your flag is: 2_primes_good_4_primes_better
This is what they gave us: '2_primes_good_4_primes_better'
lineReceived print      Goodbye!
This is what they gave us: 'Goodbye!'
~~~~~~~~
