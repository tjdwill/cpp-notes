# About Licensing

For a topic so widely discussed and debated on the internet, it seems most people in the
conversation don't know much about software licenses. Many converations are haphazard and
second- or third-hand accounts, so it's difficult to truly know what is or isn't allowed
with various licenses.  Therefore, I'm going to try to read and understand various common
licenses myself.

Now, a lot of this information comes from the Free Software Foundation (FSF), itself or
affliates, so it will have a positive spin on intent and moral quality of these licenses.
I'm sure the information is accurate, but the presentation could have implicit (maybe even
explicit) bias.

## What is *free* software?

Per [copyleft.org](https://copyleft.org/guide/comprehensive-gpl-guidech2.html#x5-40001),
free software deals with freedom, not price. The Free Software Foundation totes four
fundamental freedoms for software to be labelled free:

- Free to run (for any purpose)
- Free to change
- Free to copy and share
- Free to distribute changes.

## GNU General Public License (GPL)

From what I've seen, this is the license that causes the most fear on the internet. The GPL
(v2 and v3) are *copy-left* licenses, meaning that any work that uses GPL-licensed code
becomes itself GPL-licensed. Due to this virality, commercial/proprietary entities are
strictly against using software with this license. That doesn't mean *I* have to be though.

The idea of the GNU license is that if you, the user, have guaranteed rights associated
with some work, other users of the work (or derivative work) have the same rights. 

### GPLv2

[Read the full license here.](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html)

Section Summaries 

0. You have the right to run the Program for any purpose. The "Program" is any GPL'd work
or work based on (*i.e.* containing [un]modified aspects of) said work. The license is only
concerned with copying, distribution, running, and modification. Anything else is outside
of its scope.

1. You can copy and distribute verbatim copies of the Program in any medium, provided that
you place the following on each copy in a conspicuous way: 
    - a copyright notice
    - a disclaimer of warranty (basically "user beware, I'm not responsible for problems"
      in legalese)
    - a copy of the Program's license.

This section also establishes that you can charge distribution and warranty fees at your
discretion. This is an explicit signal that distinguishes the "free" the GPL intends
(*libre*) from the "free" many people interpret it as (*gratis*).

2. You can modify the Program (thereby forming a work based on the Program).  This work may
then be distributed and copied under Section 1's terms (copyright notice, warranty
disclaimer, and GPL License inclusion) with the addition of other required conditions:
    1. Modified files must carry prominent notices of change and include the date of any
    such change. (would access to the Git repo suffice?)
    2. Any work containing or derived from even a part of GPL'd work must itself be
    licensed as GPL *if said derived work is distributed or published*.
    3. If the work is interactive (ex. a shell), it must display or print an announcement
    of warranty (disclaimed or otherwise) and copyright notice that also informs the user
    that they may redistribute the program under the same conditions and where they can
    find a copy of the license. This notice can be omitted if the original Program 1.) is
    interactive and 2.) does *not* include such an announcement.

It's important to understand: for any work `X` that is subsequently combined with GPL'd `Y`
in some way (ex. static or dynamic linking) such that some new `X+Y` work is created, `X+Y`
is GPL'd. `X` does not have to be GPL'd *unless it itself was licensed as GPL* either
electively or via copyleft coercion. The section also stipulates that this licensing is
done at no charge. Meaning, you can't charge for the GPL license itself. You *can* charge
for distribution and/or warranty of the work (*à la* section 1).

3. You can create binary versions of GPL'd software under the terms of Sections 1 and 2,
   provided that the source is also made readily available. 

The easiest way to do this is to distribute the source along with the binary or offer both
from the same designated place (ex. GitHub, SourceForge, official website, etc.). Note that
the distribution of source does not *have* to be electronic; it could be via a physical
medium (USB, floppy, etc.  Note that the distribution of source does not *have* to be
electronic; it could be via a physical medium (USB, floppy, etc.), but who has time for
that? Post it online. Read [section 5.2.2 of CopyLeft for the legal
technicalities](https://copyleft.org/guide/comprehensive-gpl-guidech6.html).

Also note that "source" in this context means all source code of the binary including
scripts used to compile it. Basically, the intent seems to be that user of reasonable
skill/knowledge should be able to build the binary themselves.

4. Violating this License terminates the violators rights under said license (not
   third-parties who have received the violator's work but are in full compliance).
5. You don't have to accept the terms of the GPL to use GPL software. The license kicks in
   when you do copyright-related things (modify, redistribute, etc.)
6. You aren't responsible for enforcing compliance by third parties. With each
   redistribution, the receiver automatically receives license from the redistributor to
   perform the copyright-invoking actions as defined by the terms of the license.
7. Legal judgments don't excuse you from the conditions of this license. If some judgment
   renders you unable to comply to the GPL, then you can't redistribute the Program.
8. Original copyright holder can add geographic-exclusions to handle international
   copyright and patent problems.
9. The Free Software Foundation can revise or write new versions of the GPL. These versions
   are distinguished via version number. If the Program doesn't specify a version number
   (just GPL), you can choose which version applies. If it specifies a version number or
   later ("GPLv2 or later"), you can follow the terms of that version number or a later
   one.
10. You can write to the authors of a Program of interest to negotiate permission to use
    GPL-licensed software under a different distribution license.
11. Unless otherwise stated, GPL software has no warranty.
12. No liability.

### GPLv3

[Read the full license here.](https://www.gnu.org/licenses/gpl-3.0.html)

This one is has a lot more "legalese" (which means, among other things, multiple subordinate clauses with little to no
punctuation between them).

0. Definitions. The terms least intuitive in this context are:
    - *propagate*: behavior regarding a work that, without permission, would be copyright
      infringement (except when modifying for personal use or running the Program on a
      computer).
    - *convey*: propagation that enables other parties to make or receive copies.
1. Source-related things. Source code is the preferred form of the work for making
   modifications to it (i.e. text files). Object code is any non-source form.
   Other definitions in the section deal with interfaces, system libraries, and code needed
   to run the object code and modify the work (*Corresponding Source*).
2. Rights under GPLv3 are valid for as long as the copyright for the work is valid and
   cannot be revoked as long as the licensee is in good standing of its requirements.
   You're free to run the unmodified Program as you wish. You can make, run, and propagate
   (modify, etc.) GPLv3'd works without conditions as long as they are not conveyed. If
   conveyed, you must obey the terms of the license.
3. You can't use GPLv3'd software in systems that use technological measures to
   restrict user freedom of modification. If you *do* convey a covered work, you have no
   legal recourse to prevent users from jailbreaking your technological measures to assert
   rights granted to them under the License.

My interpretation could be off a bit (or just imprecise), so check [copyleft (9.5, 9.6) for further
elaboration](https://copyleft.org/guide/comprehensive-gpl-guidech10.html#x13-780009.4).

4. This is essentially Section 1 of the GPLv2. When conveying verbatim copies of the work,
   keep all legal notices regarding copyright, warranty, and license-specific information
   intact, and give provide a copy of the License with every distribution. You can charge
   any price (or no price) for each copy and you may offer support services or warranty
   protection for a fee if desired.
5. A more precise version of GPLv2 Section 2 (GPLv2§2).
    - Modified files need not be marked as modified themselves, but the total work must
      have prominent notices informing users that it is a modified work with a relevant
      date of modification.
    - Must have prominent notice that the work is released under this License and any
      additional conditions defined in section 7. 
    - The work must be licensed as a whole under this license to anyone who possesses a
      copy. 
    - Appropriate Legal Notices (copyright, warranty disclaimer, notification of rights,
      license copy information) on interactive interface-based work, unless the original
      omitted them. Mere aggregates (ex. a ZIP of GPL'd software) need not be licensed under the
      License.
6. Make life easy for yourself by making the Corresponding Source available at the same time
   and place the object code is available.
    - Unlike GPLv2, you're only obligated to fulfill CCS requests from parties in possession
    of the object code. You are not obligated to fulfill it for just any arbitrary third
    party.
    - Read the section for information on handling CCS for peer-to-peer object code conveyance.
    - Information regarding User Products (consumer products) and obligations to them. This
      seems more relevant for embedded devices, so I'll circle back once I reach that stage
      of programming experience.
**TODO**: Flesh out section on User Products (see copyleft 9.9.2)
7. This section is about addition permissions and additional requirements copyright holders
   can add to the license. It also explains when users can disregard said add-ons. The
   LGPLv3 is an example of such additional permissions.
8. This section talks about termination of rights and how to reinstate rights if
   terminated.
9. You aren't required to accept the GPLv3 to use or receive the licensed Program, but you
   automatically consent if you propagate or modify it.
10. You automatically receive a license from the original licensors when you receive a
    conveyed covered work.
11. Patent things. I'll read this later. 
**TODO**: Read the patent sections (Copyleft 9.14)
12. Same idea as GPLv2§7. If some obligation prevents you from complying with this license,
    don't convey anything that's under this license.
13. GPLv3 - AGPL collaboration.
14. FSF can revise the GPL. Information is provided on how to communicate the acceptable
    license version(s) for the work.
15. Disclaimer of Warranty
16. Limited Liability
17. How courts should interpret §15 and §16 if the judicial structure cannot effect them in
    full.


This seems like a fine license. Why do so many people seem afraid of it? I understand
companies, but independent developers? Maybe I'm missing a crucial detail or technicality
that would blindside me, but I'm not so sure.


## LGPL
