# Bug Bounty Tips #1

First video of the bug bounty tips series [[Youtube](https://www.youtube.com/watch?v=1PAS_FJ80gk)]

## Google Dork for website patterns

[Tweet](https://twitter.com/bughunterlabs/status/1490860571440590850) by [@bughunterlabs](https://twitter.com/bughunterlabs)

Find some lesser known subdomains of a target with a simple trick: Look for repeating patterns in the websites, such as a copyright or an association statement, for example in the footer.
If we go to [www.af.mil](https://www.af.mil/), we can see at the bottom “Official United States Air Force Website”. If we [search for this pattern in Google with quotation marks](https://www.google.com/search?q=%22Official+United+States+Air+Force+Website%22) we can for example discover [afcivilaincareeers.com](https://afciviliancareers.com). A website which we otherwise might not have discovered.