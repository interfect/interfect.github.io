# Stop Using `Acquire::BrokenProxy`

Ubuntu is a very popular Linux distribution. It uses `apt` and/or `apt-get` to download and install packages. Sometimes, this fails.

If this happens to you, you may be [advised](https://stackoverflow.com/a/66523384) by some presumably well-meaning person to try setting the `Acquire::BrokenProxy` configuration option to `apt`, either with something like `-o Acquire::BrokenProxy="true"` or with an `Acquire::BrokenProxy=true;` line in a configuration file, and always in conjunction with other options.

## Do Not Listen to These People

As of today, **`Acquire::BrokenProxy` does nothing**. It has done nothing since 2011 at the very latest. If you look it up in the `apt.conf` man page, it is not there. If you look for it in the `apt` source code, it is not there. It's not quite [using a regular expression to p̸͇͂͝å̵̗͇̭̻ȓ̷͇̀s̴̱̦͑͝ę̷̻͚̹̈́ ̶̥̘͊̑̊H̷̨̠̊́T̷͇͕̞̰̍M̶̨̓̓L̵̨̩͍̎͑̎](https://blog.codinghorror.com/parsing-html-the-cthulhu-way/), but it most definitely will not solve your problem and should never be used by anyone. Other options you see used alongside it might solve your problem, but this one won't.

## How Did This Happen

For one brief, shining moment, between 10:59 PM UTC on Friday, April 1st, 2005, [when it sprang fully formed from the head of Michael Vogt](https://github.com/Debian/apt/commit/284c8bbc764fed53bd0f9bc6a42a7521a0c617ce#diff-82b25fd461248155bff763f8f8d6e677191add5b7fa2c85949a6fb1c09c93208R343-R348), and 8:38 PM UTC on Monday, February 20th, 2006, [when he reluctantly had to put it down](https://github.com/Debian/apt/commit/75dd8af14b76bb84a69a927ecae93f60600b9667#diff-82b25fd461248155bff763f8f8d6e677191add5b7fa2c85949a6fb1c09c93208L343-L348), setting `Acquire::BrokenProxy` would force APT to always re-download the signature file for a package repository, instead of just asking to be sent it if it had been modified. Presumably this is because some "broken" proxies were just sending the old version of the file even when it has changed.

This is all it has ever done, as far as I can tell. You can `git clone https://salsa.debian.org/apt-team/apt.git` and `git log -S "BrokenProxy"` in there to find commits that add and remove this string to/from the APT source code; I only count the two.

This feature first came out in APT version `0.6.35ubuntu1` and would have last been available around APT version `0.6.43.3`, going by the state of the changelog file at those commits. Digging through the `Packages.gz` files on http://old-releases.ubuntu.com, it looks like it should have had an effect on **Ubuntu 5.04 (Hoary Hedgehog)**, **Ubuntu 5.10 (Breezy Badger)**, and **Ubuntu 6.06 LTS (Dapper Drake)** (which has `0.6.40.1ubuntu9`). Version 6.06 was an LTS release that went out of support in 2011. After 2011, or on any Ubuntu that is 6.10 (Edgy Eft), with apt `0.6.45ubuntu14`, or later, nobody has had any business whatsoever using this option.

However, `Acquire::BrokenProxy` remains a popular `apt.conf` option to this day. It got its humble start in December 2005, when its author suggested it as a troubleshooting approach in [an Ubuntu bug report about BADSIG errors](https://bugs.launchpad.net/ubuntu/+source/update-manager/+bug/24061#yui_3_10_3_1_1652226092361_282). Over the next few years, many lost souls found their way to this bug report, and a few tried passing `-o Acquire::BrokenProxy=true` to their package managers, following an encouraging 2006 report by a user that it had worked for them. Despite a couple of subsequent reports that the option did not (and, depending on Ubuntu version, most likely could no longer) help, in 2009-2010 the option made the rounds on [blogs](https://web.archive.org/web/20091212164529/https://www.ubuntugeek.com/how-to-fix-the-ubuntu-gpg-error-badsig.html) and [forums](https://www.linuxquestions.org/questions/linux-newbie-8/hash-sum-mismatch-debian-781878/), and by 2012 it had been posted back on the BADSIG bug as part of ["the fix"](https://bugs.launchpad.net/ubuntu/+source/update-manager/+bug/24061#yui_3_10_3_1_1652226092361_872) for a misbehaving proxy.

## Recombination and Dockerization

In 2013, two major technologies arrived on the scene that changed computing forever: the Docker container management system, and [the `99fixbadproxy` APT config file](https://gist.github.com/trastle/5722089/revisions):

```
Acquire::http::Pipeline-Depth "0";
Acquire::http::No-Cache=True;
Acquire::BrokenProxy=true;
```

Soon, people began combining the power of these two technologies, presumably because Docker image building is an `apt` stress test and the other two options actually do something useful. Today, the `99fixbadproxy` file, with this option **that has never had any effect in any Ubuntu version that has ever been available as a Docker image**, [can be found in Dockerfiles all over Github](https://github.com/search?l=Dockerfile&q=99fixbadproxy&type=Code). It has even ascended to become part of Github itself: if you use Github Actions to run tasks on Ubuntu, [your Ubuntu machine will be configured with `Acquire::BrokenProxy` by a `99fixbadproxy`-descendant](https://github.com/actions/virtual-environments/blob/1457fb6402aaad8508d16a8968106b628368867f/images/linux/scripts/base/apt.sh#L17-L22).

## Conclusions

Truly, it is impossible to underestimate the power of the `Acquire::BrokenProxy` option. It's completely inert, it comes attached to other code that may or may not help resolve an intermittent failure, and it looks plausible enough that hardly anyone, present company excepted, has ever thought to question its presence. I can't help but think of the humble transposon, being copy-and-pasted and copy-and-pasted, pretending to be a gene, [until it's 10% of the human genome](https://en.wikipedia.org/wiki/Alu_element).

Remember this next time you take a piece of code from the Internet without understanding the justification for all of its parts. Remember it especially when you write a justification for code that you do not know for a fact to be true, all the way down the stack. Someday, [a Programmer at Large will thank you for letting them get back to _Lesbian Space Pirates_ a little sooner](https://archiveofourown.org/works/9233966?view_full_work=true).
