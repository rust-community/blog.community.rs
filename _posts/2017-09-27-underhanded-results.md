---
layout: post
title: "The 2016 Underhanded Rust Contest - The Results"
author: Jan-Erik Rediger, Erick Tryzelaar
description: "The Rust Community Blog is open for business!"
category: underhanded
lang: en-US
---

## The Underhanded Rust Contest - The Results

Back in December last year we launched the first annual [Underhanded Rust
Contest]({% post_url 2016-12-15-underhanded-rust.en-US %}). Our goal with
[Rust](https://www.rust-lang.org/) is to make it easy to write trustworthy
low-level software that is resistant to accidental security vulnerabilities.
This contest was to test Rust's ability to protect against *deliberate*
vulnerabilities in the face of scrutiny.

The Rust Community Team is now pleased to announce the winner of the contest.
We received a handful of submissions, all exploiting tiny flaws that might slip
through a code review. One of Rust’s goals is to make it easy to write
trustworthy low-level software that is resistant to these accidental security
vulnerabilities.

This year’s challenge involved a minimal banking and money transfer
application, providing a simple HTTP API:

## The Challenge: Salami Slicing

The startup you work at, Quadrilateral, just pivoted into the payment
processing market, and you've been tasked to implement the backend.
Unfortunately for them, you are already burnt out from all these late night
pivots and broken promises. You’re ready to split, but before you leave, you
figure it’s time to make the company pay for all that overtime they owe you.
Your challenge is to:

* Create a simple web server that supports at least creating accounts and
  payment submissions.
* Payment transactions should at least include an account, a customer, and a
  payment amount.
* The Underhanded Part: quietly carve out fractions of a penny from each
  transaction into an account you control (otherwise known as the salami
  slicing scam), without that being obvious from the source.

We didn't restrict which web frameworks to use and gave candidates the freedom
to design their API to their liking. An example implementation of a
non-vulnerable [Quadrilateral bank
application](https://github.com/erickt/quad-bank) was provided to ease the
start.

## Runners up

We had 4 submissions, two from Aidan Hobson Sayers, and two from serejkaaa512,
all of which contained subtle flaws to carve out fractions of a penny from each
transaction.

Thanks to both candidates and their work! (The order of the following list is
not meant as a placement of the different entries. We valued all 4 entries and
decided to only name the final winner).

### Aidan Hobson Sayers

The full implementation is [available online](https://github.com/rust-community/underhanded-submissions/tree/master/2016/aidanhs/submission2).

> Exploit
> =======
> 
> In an interesting twist, this exploit may be more obvious to users who aren't
> so familiar with Rust and/or have experience in other languages - it actually
> uses data races to do its thing.
> 
> "But wait!" I hear you cry, "there are no data races in safe Rust, and I see
> no unsafe in your code!". This is why I say the issue may be more discoverable
> for people who perhaps don't take Rust promises for granted :)
> 
> The problem is that no data races is not *quite* true thanks to a known bug [^4]
> that's had remarkably little attention, likely partly because it's not
> immediately obvious what damage you can do and partly because fixing it would
> break quite a lot. After thinking about it for a while, I still couldn't
> come up with many places where it would do terrible things - you can do a
> "I `mem::replace`d a bit of data and I then accessed a field on it which gave
> me an unexpected value because the borrow checker didn't warn me", but there
> are variations of this 'problem' in safe code as well [^5], and it just gives you
> an unexpected but consistent result. I wanted something better than that, to
> make sure the slicing was more subtle and can be activated for just me.
> 
> In the end, I concluded that (as far as I could see, which may not be very far!)
> the issue only gets interesting if you have a) noalias data being generated
> *within* functions [^6] or b) data races. Since a does not exist in Rust right now
> it had to be b. So, what kind of mutation could we justify? Remember, all that
> can be done is change where the reference points to, so memory safety bugs are
> not easy. In the end I decided that a `mem::replace` of a struct with some
> sentinel value would be easiest (and justifiable since the code would be
> significantly uglier if all the `UserAccount`s in the DB were in `Option`s).
> However, I needed some way of getting the `&mut` reference into the thread and
> only doing the `mem::replace` in there (otherwise it's not racing) - this is
> where the `crossbeam` scoped threads api comes in (totally safe...assuming no
> bugs in `rustc`), permitting the `&mut` reference to be sent to another thread.
> 
> So there's a data race in place, all that's left is to find a way to slow down
> the timings in a way that lets me perform the exploit in a repeatable way. I
> happen to know that fraud checks can want to talk to external services, so
> the fake delay inserted is not unrealistic for an real implementation. Rate
> limiting was then added to have some timing I can remotely control, useful for
> reacting to changes in response time of the fraud service.
> 
> The whole overdrafts thing is just to justify using `i64` just so the exploit
> works in debug mode. Note that it'd also work fine in release mode with `u64` due
> to defined wrapping behaviour.
> 
> Summary
> -------
> 
> The good:
> 
>  - Works on 1.15.0 stable upwards (and probably downwards)
>  - No `unsafe`, mostly defensible design decisions
>  - Clippy has nothing to say
>  - Fully dynamic and remotely controllable exploit (no hardcoded accounts)
>  - Scratches off bullet point 4 of rust-lang.org! ("threads without data races")
> 
> The meh:
> 
>  - Relies on a known soundness bug (though severity is perhaps underestimated
>    at the moment)
>  - Relies rather heavily on not questioning the `transfer_charge: -1.0` in the
>    fake currency. I think it may be possible to come up with something with the
>    infinities and NaN, but it'd look more suspicious.

### serejkaaa512's First Submission

Find the submission [here](https://github.com/rust-community/underhanded-submissions/tree/master/2016/serejkaaa512/submission1).

Macros export system allows to overwrite the standard macros. Exploit replaces
standard macro `format!` with the poisoned one, hiding it deep in some vendored
dependency. serejkaaa512 goes into more detail in
[this blog post]({% post_url 2017-09-27-underhanded-serejkaaa512-submission1 %}).

### serejkaaa512's Second Submission

Find the submission [here](https://github.com/rust-community/underhanded-submissions/tree/master/2016/serejkaaa512/submission2).

Exploit is based on Rust importing modules system. Rust has a `use` keyword,
which allows to import names into local scope. Additionally imports can be
renamed using the `as` keyword. The exploit uses these two keywords to use a
trait from one module and all other things from another. serejkaaa512 goes into
more detail in [this blog post]({% post_url
2017-09-27-underhanded-serejkaaa512-submission2 %}).

## The Winner: Aidan Hobson Sayers

Aidan submitted two different underhanded implementations of the Quadrilateral
bank application, but in the end the jury awarded the first place to his first
submission. The full implementation is
[available online](https://github.com/rust-community/underhanded-submissions/tree/master/2016/aidanhs/submission1).
Before reading any further, check out the code and see if you can find the
vulnerable part.

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

Aidan provided a detailed write-up of his entry:

> I was maybe a bit heavy-handed with the comment saying "look! Protection
> against dir traversal" and the tests trying to convince you of the same thing -
> this is indeed a classic directory traversal attack, with two extra pieces
> to turn it into a working exploit.
>
> The 'issue' is that `Path.join` will throw away your existing path if the
> path you're joining with is absolute. So although canonicalize-and-compare
> does protect against `..` and symlinks, it doesn't help against
> `Path::new("a").join("/etc/passwd")` [^0]. I actually made this mistake
> recently [^1] so I personally think it passes as an innocent error!
>
> Directory traversal isn't very useful by itself, especially if you want to
> alter code flow (e.g. salami slice) rather than just leak data, so I combined
> this with 'accidentally' overwriting keys in HashMaps without checking for
> conflicts [^2] - justifiable if you're assuming that all the
> JSON is trusted. In terms of the execution of the program, having currency
> aliases unconditionally replacing other keys in the `CURRENCIES` hashmap lets
> you pick the account to receive transfer charges for any currency you like -
> invisible to the end user and choosing lesser-used aliases ('$' vs 'USD')
> probably lets you escape the notice of Quadrilateral.
>
> This brings me to the final (and the most entertaining to me) part - so far
> we can pick a path, and if there's valid data in the path then we can get it
> loaded into the application and make it give us money. But where can we get
> data from? Hardcoding (e.g. some example files in a 'tests' directory) would
> work, but was too static and easy to spot - I wanted something dynamic,
> remotely controllable and isn't as obvious as "please keep using this crate I
> own so I can create files on disk". If you've not already figured it all out,
> you might want to stop here and have a think about how the crucial final step
> might work! (note that `script.sh` just makes requests via curl, there's no
> file creation in it)
> 
> ----
>
> It turns out there's one data source that we can rely on being on any machine
> used to build Rust projects - the crates.io index! I crafted the `Currency`
> and `CurrencyFeatures` to match up to the crates.io JSON, e.g.
>
 > - the (oddly named) `features` field
 > - using `Vec<String>` for `transfer_charge_accounts` and making up a
 >   justification to do with account rotation (rather than a simpler single
 >   u64) because that's what [crates.io emits](https://github.com/rust-lang/crates.io/blob/7f731d4/src/git.rs#L21)
>
> This was combined with some misdirection with `Optional<_>` to distract from
> the suspicious crates.io-like field naming, relying on the helpful behaviour
> of `serde_json` to ignore fields it's not expecting. Bors [faithfully
> committed](https://github.com/rust-lang/crates.io-index/commit/7478a2083899402e1b2ed44287b194c06ada748b)
> my JSON, and it's primed for slicing salamis [^3]. If I wanted to start
> skimming off other currencies, I just need to publish another crate and wait
> for Quadrilateral to inevitably update their crates.io index.
>
> Summary
> -------
> 
> The good:
> 
>  - Works on 1.15.0 stable upwards (and downwards, if I replace serde derive
>    with codegen)
>  - (Mostly) explicable as innocent error in programming (there's even a test
>    of the buggy area to show willing!)
>  - Clippy has nothing to say
> 
> The meh:
> 
>  - I could demonstrate variations of this bug in any language...I
>    guess the lesson is that Rust doesn't (can't) help with bad path handling
>  - Requires a crates.io index on the machine where this runs. Possibly not as
>    unlikely as you might think, especially given there's a Dockerfile the
>    company can use with zero effort

## The Jury’s Comments

> this exploited a relatively common path manipulation bug to extract an
> exploit package from a completely novel place, the crates.io index. This
> exploit was fully dynamic and remotely controllable. Neither the compiler and
> Clippy had anything to say about the bug, and it’s generalizable to work with
> any version of the compiler. While this exploit relies on the crates.io index
> being present on the machine where this runs, it’s surprisingly common for
> the index to be baked into systems. For example, you can find the file in
> official [Rust Dockerfile](https://github.com/rust-lang-nursery/docker-rust)
> containers if you use `cargo build` to build a crate with dependencies.

> I really had no idea how it worked, and exploited a really clever attack
> venue. I could totally see someone embedding some executable lua/javascript
> and using this trick to completely take over a system. This is a great
> argument to have crates.io sanitize its inputs, which I didn't even consider
> being an attack target in this contest.

> I initially blew this one off as a simple path traversal attack, but I think
> it's a bit more than that because of how Cargo is involved.

## Prizes

Aidan Hobson Sayers and serejkaaa512 will soon receive a limited-edition Ferris plushie:

![plushie ferris]({{ site.url }}/assets/plushie-ferris.jpg)

a rusty metal Ferris:

![iron ferris]({{ site.url }}/assets/metal-ferris.jpg)

and lots of stickers.


**Congratulations Aidan Hobson Sayers, you are the most underhanded Rust programmer of 2017.** 

---

Footnotes from Aidan:

[^0]: This behaviour is the same as in Python

[^1]: I've read the docs for both Python and Rust path joining many times, but
	unfortunately I'd forgotten them until I stumbled across the behaviour
	recently when I *did* want to join absolute paths as a suffix. It ended up
	being surprisingly tricky [to do this](https://github.com/aidanhs/machroot/commit/772507eda42c2fda#diff-639fbc4ef05b315af92b4d836c31b023R87),
	but perhaps I missed something.

[^2]: Whenever I personally use hashmaps, throwing away old entries is so
	rarely what I want to do that I pretty much always wrap inserts with
	assertions, like
	[this](https://github.com/aidanhs/ayzim/blob/e0a0b680f61b3668bf2bebfd47f961d966b72b7c/src/optimizer.rs#L1332-L1333)
	and many others in that file.

[^3]: To be explicit, I don't consider this coming anywhere near violating the
	"Do not submit patches upstream, or otherwise inject malicious code into any
	dependency in the wild" rule - this is metadata that's only possibly useful for
	this toy vulnerability contest.

[^4]: [#38899](https://github.com/rust-lang/rust/issues/38899) - it's also
	marked `I-unsound` so has likely been at least considered for submission by
	other underhanded rustaceans.

[^5]: Trivial example:

    ```rust
    let mut x = ...;
    foo(mem::replace(&mut x, ...));
    bar(x.a); // not the x you were looking for
    ```
    As you can see, this isn't a great basis for an exploit.

[^6]: A brief background: alias annotations were previously generated on
	function arguments, but were disabled because they were buggy. You can follow
	the chain of issues downwards from
	https://github.com/rust-lang/rust/issues/31681 if you're interested. However,
	in addition to argument alias analysis, there's scoped alias analysis data
	(https://github.com/rust-lang/rust/issues/16515) which is likely to bring all
	sorts of new miscompilation opportunities with it.  This latter set of metadata
	is what I'm referring to, though I didn't fully think through whether the
	exploit would be possible with it - just suspicions.
