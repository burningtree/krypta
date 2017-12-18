# Krypta

Generating random bits, passwords, recovery phrases and Bitcoin private keys / addresses (including QR codes) from text seed and salt. Deliberately implemented in single file pure LuaJIT (including crypto routines and hashing functions) without any external dependencies. This is useful for generating passwords / private keys etc for the most important stuff you have. This software is ugly, not user friendly and not meant for computer illiterate. If you don't understand what is written in this document, then Krypta is very probably not for you and using it might be dangerous.

Run it without arguments to see the basic explanation of options.

Krypta is a less traditional way to keep your important passwords / codes / wordlists safe. Extremely important things. Like the private keys from your 100 BTC Bitcoin address. Instead of keeping this info in the physical safe or hardware device (e.g. Trezor), you have to remember one long super-secret passphrase and all your other secret info is derived from this passphrase using this script and your notes which you don't have to keep especially secret.

You might think this sounds dangerously similar to the infamous "memory wallets" but bear with me.

When you start Krypta, you enter your secret passphrase which you have to remember exactly and never forget. This passphrase should have sufficient entropy to generate 256 bits checksum and not be something that can be found using brute force or dictionary attacks. Note that this definition if rather vague and it's not easy to evaluate the suitability of any given passphrase. But it's very very important to choose something that cannot be easily guessed / cracked (and, again, "easily" is very vague term here). One acceptable way MIGHT BE for example to take firt 50 characters from the middle of your favorite song, for example "NoStopSignsSpeedLimitNobodysGonnaSlowMeDownLikeAWh" (from "Highway to Hell"). In this specific example, note that 50 ASCII characters is still not enough for full 256 bits of entropy and that you have to be sure to remember whether you used "NobodysGonna...", "NobodySGonna..." or "NobodyIsGonna...". This is not an easy decision and it cannot be underestimated!

The security of your passphrase is further increased by optional Salt, which is another piece of information that pertains SPECIFICALLY TO YOU, e.g. your e-mail or your phone number or some important date. This information is not secure by itself (at all) but when Krypta combines it with your passphrase, it increases its security significantly.

The Passphrase and Salt are then used to generate your Master key, which is a 256-bit hash function. You can select the difficulty of the hash function from 1 to 31, where 1 takes a fraction of second on average computer and each subsequent difficulty take approximately twice as long (the max difficulty takes many years). You should select as high difficulty as possible for your computer because it makes any bruteforcing / dictionary attacks much, much harder if the attacker can only try 1 combination in 5 seconds instead in 1 million combinations in 1 second.

So, the resulting 256-bit Master key is uniquely defined by your Passphrase, your Salt and your Difficulty. Krypta will never actually show you your Master key (although you can see its 12-bit checksum) but it's used to calculate all other keys / passwords that want to keep using Krypta. Master key is found by running a modified XorShift PRNG (with 4 sequential LFSR registers that cause "skipping" of some PRNG values) until some specific math conditions are met (many, many times in sequence) for the generated numbers. The Master key is then taken from the following 256 PRNG values. The original unmodified XorShift PRNG period is 2^128 and the resulting period in combination with LFSR registers is something around 2^220, if I remember correctly.

After Krypta takes several seconds to calculate your Master key (which SHOULD take a while, using high difficulty), you can then (quickly) derive any quantity of secret data from the combination of this Master key and any textual index.

For any Index you enter, Krypta will immediately show you things like password, Bitcoin private key, BIP39 word sequence and other security data you might need to generate. These are different for each Index and cannot be traced back to your Master key, Index or Passphrase. Some examples of Indexes: "BTCColdStorage", "Facebook", "Mycelium" etc...

For each Index, you get full set of security data, e.g. long password, short password, uncompressed BTC private key, compressed private key, long hex number etc... To prevent clutter, you can add a Prefix to your Index (all Prefixes are displayed after you enter any non-prefixed Index). For example entering the Index "btcu:ColdStorage" is the same thing as entering just "ColdStorage" and then looking for the line that has prefix "btcu:" (uncompressed BTC key). Because Bitcoin QR codes take lots of screen space, they are displayed only when you enter specific prefix ("btcu:" or "btcc:").

In addition, all Master key / Index combinations generate two "Check words", and all Master key / Prefix / Index combinations generate three "Check words". You can use these as a checksum of sorts.

At this time, you might be overwhelmed by all this information and you might think that you can never remember all of this. But you don't have to!

The only thing you absolutely, positively have to remember, is your Passphrase. Everything else, i.e. Salt, Master key checksum, Indexes, Prefixes, Check words, can be stored in a text file which you don't have to be extremely paranoid about (of course it IS better if no one else sees it).

So, for example, you can have the following in your text file:

```
passphrase hint: 50 letters from second verse of THAT song, in CamelCase.
difficulty: 8
salt: satan@hell.org
checksum: 0x89A
Main BTC Cold storage: btcc:BTCcold, checkwords "leader author tunnel"
Facebook password: pwd40:Facebook, checkwords "penalty party ritual"
Mycelium wordlist: wrd24:Mycelium, checkwords "enlist enjoy midnight"
```

IMPORTANT: Do you see something wrong with this? The Passphrase hint in this specific example is UNSAFE and makes you much easier target for someone who sees this hint! It's all about balance of security and ease of remembering. Do your homework.

You can keep copies of this file in Dropbox, Google Drive and your desk drawer. Don't forget to update all copies when you update the file (e.g. add new Indexes). The checkwords are a good way to immediately see if you entered something wrong somewhere.

You can also use some of this data as a command line options to Krypta (and call it from your Bash script) or enter them directly into Krypta (have a look at the first few lines of code). Of course having your Difficulty or Salt in a Bash script alongside Krypta again decreases your security a little bit more.

## Practical security considerations:

* If you forget / lose your Passphrase, Salt, Difficulty **or** Indexes, you are doomed and cannot recover your secure data.
* If someone guesses and/or steals your Passphrase, Salt, Difficulty **and** Indexes, you are doomed because he can see your secure data.
* If you have keylogger in the PC you use to run Krypta, you are very probably doomed.
* If you have a virus that can see what happens in your PCs memory, you are very probably doomed.

It's best to run Krypta on a special computer (old laptop?) with minimal Linux installation and no Internet connection. That's why Krypta runs in text mode, requires only minimal LuaJIT (no dependency on any libraries / apps) and is self-contained in a single script file.
