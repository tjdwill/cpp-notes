# About Licensing

For a topic so widely discussed and debated on the internet, it seems most people in the
conversation don't know much about software licenses. Many converations are haphazard and
second- or third-hand accounts, so it's difficult to truly know what is or isn't allowed
with various licenses.  Therefore, I'm going to try to read and understand various common
licenses myself.

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

[Read the full license here.](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html).

Section Summaries -----------------

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
   provided that the source is also made readily available. The easiest way to do this is
   to distribute the source along with the binary or offer both from the same designated place
   (ex. GitHub, SourceForge, official website, etc.). Note that the distribution of source
   does not *have* to be electronic; it could be via a physical medium (USB, floppy, etc.
   Note that the distribution of source does not *have* to be electronic; it could be via
   a physical medium (USB, floppy, etc.), but who has time for that? Post it online. Read
   [section 5.2.2 of CopyLeft for the legal technicalities](https://copyleft.org/guide/comprehensive-gpl-guidech6.html).

Also note that "source" in this context means all source code of the binary including
scripts used to compile it. Basically, the intent seems to be that user of reasonable
skill/knowledge should be able to build the binary themselves.
