% TALLOW.PATTERNS(5)
% Auke Kok `<auke-jan.h.kok@intel.com>`

# tallow.patterns

Tallow pattern matching configuration files.


# SYNOPSIS

tallow(1) uses regular expressions to match journal entries and extract an IP
address from them. JSON files are used to configure the patterns and banning
thresholds used by tallow(1).

`/etc/tallow/*.json`
`/usr/share/tallow/*.json`


# DESCRIPTION

tallow(1) uses regular expressions to match journal entries and extract an IP
address from them. JSON files are used to configure the patterns and banning
thresholds used by tallow(1). This adds the ability to extend the patterns
tallow(1) will recognize. Many JSON files can exist for logical grouping. The
tallow(1) daemon will read all JSON files in the configuration directories at
startup.

tallow(1) operates with default pattern definitions
in`/usr/share/tallow/*.json`. Users can add more patterns with their own JSON
files under `/etc/tallow`. The default JSON files can be overridden by creating
the same file under `/etc/tallow`.


# FILE FORMAT

Pattern configuration files use the JavaScript Object Notation (JSON) format.

The JSON must be two levels deep and all properties are required. The root
object is an array containing objects with a `filter` key and an `items` key.

* `filter` is a string that defines a field for filtering the journal file.
  This helps make sure patterns are only matched to a subset of journal
  entries. See systemd.journal-fields(7) for valid journal fields.

* `items` is an array of objects that contains three elements: `ban`, `score`,
  and `pattern`.

   * `ban` is an integer that defines the number of seconds to ban originating
     IP for. If this value is > 0, the IP address get banned immediately when a
     journal entry matches `pattern`.

   * `score` is a double that defines a value to add to the accumulated "score"
     of an originating IP address each time a journal entry matches
     the `pattern`. If the combined score is > 1.0, tallow bans the originating
     IP for the default time of 1 hour. The `ban` element value above is not
     used for bans made due to `score`.

   * `pattern` is a string that defines a Perl Compatible Regular Expressions
     (PCRE) to match against the filtered journal entries. The PCRE should
     extract exactly one substring: the originating IP address for tallow(1).
     See systemd.journal-fields(7) for valid journal fields.


# EXAMPLES

1. The JSON below is a snippet from one of the default pattern configuration
   files for blocking certain failed `sshd` connections.

   The first pattern will ban an IP address after it fails to login 6 times
   causing it to reach a total score > 1.0.

   The second pattern will ban an IP address for 10 seconds every time a login is
   attempted with an invalid user. Additionally, it will ban the IP address for
   1 hour if it attempts to login with an invalid user 6 times causing it to
   reach a total score > 1.0.

   See the `/usr/share/tallow/sshd.json` file for more `sshd` examples.

   ```
   [
     {
       "filter": "SYSLOG_IDENTIFIER=sshd",
       "items": [
         {
           "ban": 0,
           "score": 0.2,
           "pattern": "MESSAGE=Failed .* for .* from ([0-9a-z:.]+) port \\d+ ssh2"
         },
         {
           "ban": 10,
           "score": 0.2,
           "pattern": "MESSAGE=Invalid user .* from ([0-9a-z:.]+) port \\d+"
         }
       ]
     }
   ]
   ```



2. The JSON below defines a pattern for blocking connections based on error logs
   from `nginx-mainline` if placed in a `/etc/tallow/nginx-mainline.json` file.

   The pattern will ban an IP address for 15 seconds every time it attempts to
   access a script that does not exist. Additionally, it will ban the IP
   address for 1 hour if it attempts to access invalid scripts 4 times causing
   it to reach a total score > 1.0.

   ```
   [
     {
       "filter": "SYSLOG_IDENTIFIER=nginx-mainline",
       "items": [
         {
           "ban": 15,
           "score": 0.3,
           "pattern": ".Primary script unknown. while reading response header from upstream, client: ([0-9a-z:.]+),"
         }
       ]
     }
   ]
   ```


# SEE ALSO

tallow(1), tallow.conf(5)


# BUGS

`tallow` is `NOT A SECURITY SOLUTION`, nor does it protect against random
password logins. An attacker may still be able to logon to your systems if you
allow password logins.
