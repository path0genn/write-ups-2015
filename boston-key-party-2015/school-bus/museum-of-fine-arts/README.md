# Boston Key Party CTF 2015: museum-of-fine-arts

**Category:** 
**Points:** 
**Solves** 
**Description:**

> Because cryptography is hard, we only implemented a hand-made PRNG. What could possibly go wrong? : 25

## Write-up

Museum of Fine Arts
Level 4 of the BKP CTF, worth 25 points. 

The challenge:
The goal here is to work with the hand-made psuedo random number generation in some way. The source code for the challenge page is:

-----------------------------------------------------------------------------------------------------
1  <html>
2  <head>
3      <title>level4</title>
4      <link rel='stylesheet' href='style.css' type='text/css'>
5  </head>
6  <body>
7
8  <?php
9  session_start(); 
10
11 require 'flag.php';
12
13 if (isset ($_GET['password'])) {
14    if ($_GET['password'] == $_SESSION['password'])
15        die ('Flag: '.$flag);
16    else
17        print '<p class="alert">Wrong guess.</p>';
18 }
19
20 // Unpredictable seed
21 mt_srand((microtime() ^ rand(1, 10000)) % rand(1, 10000) + rand(1, 10000));
22 ?>
23
24 <section class="login">
25        <div class="title">
26                <a href="./index.txt">Level 4</a>
27        </div>
28
29		<ul class="list">
30		<?php
31		for ($i=0; $i<3; $i++)
32			print '<li>' . mt_rand (0, 0xffffff) . '</li>';
33		$_SESSION['password'] = mt_rand (0, 0xffffff);
34		?>
35		</ul>
36
37      <form method="get">
38                <input type="text" required name="password" placeholder="Next number" /><br/>
39                <input type="submit"/>
40        </form>
41 </section>
42 </body>
43 </html>
-----------------------------------------------------------------------------------------------------

There are a few important things going on that we need to look at in order to get the flag:
1. A new PHP session is established each time the page is loaded (line 9).
2. The random number generation function mt_rand() is seeded by mt_srand() at line 21.
3. mt_srand() seeds mt_rand() by using the cryptographically insecure microtime() function.
4. At line 31, a for loop prints out 3 random values produced by mt_rand(). We can use these values to get the flag.
5. At line 33, the password we enter is compared to the 4th random number produced by mt_rant() for this specific PHP session.

In order to predict what mt_rand() will produce as the 4th random number, we need to know the seed produced by mt_srand(). Why does this work? PHP mt_rand() produces predictable numbers, and if you know the seed, you can predict output. So how can we get the seed? We need to know the value of microtime() passed to mt_srand(), then we can brute force the seed. By default, microtime() returns a string in the form "msec sec", where sec is the number of seconds since the Unix epoch (0:00:00 January 1,1970 GMT), and msec measures microseconds that have elapsed since sec and is also expressed in seconds. We can get one of these values from the CTF server by starting a packet capture, and refreshing the challenge page. Looking inside the packet capture, you will see the server timestamp returned in the HTTP header. Using the returned date, you can determine the number of whole seconds that have passed since Unix epoch. In my case, sec = 1425144136. Now you are asking, what about the microseconds piece? For this, we can brute force all 1 million possible microsecond values. I used the following PHP code to generate the 4th random number and get the flag. My $result is simply the first random number provided to me on the challenge page.

1  $sec = 1425144136;
2  for ($msec = .00000000; $msec < 1; $msec += .00000001)
3  {
4  	$microtime = (string)$msec . " " .  (string)$sec;
5	mt_srand(($microtime ^ rand(1, 10000)) % rand(1, 10000) + rand(1, 10000));
6	$result = mt_rand(0, 0xffffff);
7	if ($result == 14278054)
8	{
9		print "SUCCESS";
10		print $microtime . "\n";
11		print mt_rand(0, 0xffffff) . "\n";
12		print mt_rand(0, 0xffffff) . "\n";
13		print mt_rand(0, 0xffffff) . "\n"; //winning number
14		}
15  }

path0gen

## Other write-ups and resources

* none yet
