# Maildirarc #

## Introduction ##

Maildirarc is a tool for archiving a given set of Maildir folders.
It can archive them to other Maildir folders, or to mbox files. It
has options to tailor the exact behaviour to hopefully suit most
mail setups.

Maildirarc is written in Perl and requires the following libraries:

 * Getopt::Long
 * File::Copy
 * Date::Parse (part of the TimeDate CPAN module)

The first two are usually installed with Perl.

## Usage ##

Run `maildirarc` to view the usage information:

    Usage: maildirarc [-h] [-f folder] [-m mbox] [-d days] [-r] [-c] [-n] folder [folder...]
    Archives the contents of the given folders.

      -f    the full path of the archive folder [default: [folder].archive]
      -m    the full path of the archive mbox
            (%y substituted for year, %m month, %n for folder name)
      -d    the age, in days, at which to archive a message [default: 90]
      -r    REMOVE the messages rather than move them
      -c    copy rather than move messages (leaves original intact)
      -n    don't do anything; just say what would be done
      -h    print longer help information

Note that Maildirarc only looks in the `cur` folder of a given
Maildir folder. This means anything in `new` will be ignored. Since
Maildirarc is primarily an archival tool it doesn't make sense to
archive stuff that hasn't been seen yet.

## mbox Format ##

There are various mbox formats in existence, so I had to choose
which one to use for Maildirarc. I decided on the variant known as
mboxcl. The mboxcl format quotes lines in the message body that
start with `"From "` and additionally adds a `Content-Length` header.
This is consistent with the behaviour of Mutt.

Maildirarc needs to create the `"From "` separator line for each message
when adding it to an mbox file. To do this it requires the delivery date
and the sender's email address. It obtains these as follows:

 * The delivery date is taken by looking first at the `Delivery-date`
   header. If that doesn't exist it next looks at the latest `Received`
   header. It never looks at the `Date` header since this may be bogus or
   invalid. If a date cannot be parsed or isn't found at all Maildirarc
   will abort.

 * The sender's email address is obtained by looking at these headers
   in order: `Return-path`, `From`, `Sender`, `Reply-To`. If it fails
   to find an address in any of those it aborts.

## Examples ##

Some example uses of Maildirarc taken from the `maildirarc --help`
output:

    # Archive messages older than 30 days in /home/tdb/Maildir
    # to the Maildir folder /home/tdb/Maildir/.old
    maildirarc -f /home/tdb/Maildir/.old -d 30 /home/tdb/Maildir

    # Archive messages older than 90 days for both of these folders
    # to a sub-folder (courier style) called archive
    maildirarc -d 90 /home/tdb/Maildir/.lists /home/tdb/Maildir/.work

    # Test to see what would be archived with the default settings
    # in the folder /home/tdb/Maildir/.personal
    maildirarc -n /home/tdb/Maildir/.personal

    # Archive messages older than 30 days in the Maildir folder
    # /home/tdb/Maildir/.spam to the mbox file called
    # /home/tdb/MailArchive/spam
    maildirarc -d 30 -m /home/tdb/MailArchive/spam /home/tdb/Maildir/.spam

    # Remove messages older than 15 days from /home/tdb/Maildir/.vspam
    maildirarc -d 15 -r /home/tdb/Maildir/.vspam

    # Make a complete copy of /home/tdb/Maildir/.folder
    maildirarc -d 0 -c -f /home/tdb/Maildir/.foldercopy /home/tdb/Maildir/.folder

## Potential Issues ##

 * There is currently no locking. This is a particular issue when
   archiving to mbox files that may be accessed by other applications.
   Maildirarc's current behaviour keeps mbox files open until it
   exits which may further compound the issue.

 * Maildirarc keeps mbox files open whilst running. This could mean
   a file descriptor limit is hit. This method of operation may be
   reconsidered.

 * Maildirarc escapes lines starting with `"From "` by adding a `>`
   character at the start. It also adds a `Content-Length` header
   which might make the `"From "` escaping redundant, depending on
   the MUA being used. However, I felt it was best to do both methods
   for maximum compatibility.

 * The format of messages in mbox files has been deduced by looking
   at RFCs and examining the behaviour of other programs, in
   particular Mutt. The files haven't been extensively tested with
   other applications, although problems are not anticipated.

 * If Maildirarc finds any of the following headers when converting
   to mbox format it'll throw them away and replace them with our
   generated ones. The reasoning for this is that we know ours to
   be correct by examining the message and Maildir flags making it
   safe to replace them. Use the `--debug` flag to see what the
   differences in the headers are.

        Status:
        X-Status:
        Content-Length:

 * Maildirarc originally added a Lines header to messages when
   converting them to mbox format. However, after some web searching,
   and reading of RFC 5536, I concluded these were used for news
   rather than email, and even there they're now deprecated. So now
   Maildirarc completely ignores the Lines header.

## License ##

Maildirarc is released under the GPLv2 license. See the COPYING
file for details.
