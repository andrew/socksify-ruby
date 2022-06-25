SOCKSify Ruby
=============

What is it?
-----------

**SOCKSify Ruby** redirects any TCP connection initiated by a Ruby script through a SOCKS5 proxy. It serves as a small drop-in alternative to [tsocks](http://tsocks.sourceforge.net/), except that it handles Ruby programs only and doesn't leak DNS queries.

### How does it work?

Modifications to class `TCPSocket`:

*   Alias `initialize` as `initialize_tcp`
*   The new `initialize` calls the old method to establish a TCP connection to the SOCKS proxy, sends the proxying destination and checks for errors

Additionally, `Socksify::resolve` can be used to resolve hostnames to IPv4 addresses via SOCKS. There is also `socksify/http` enabling Net::HTTP to work via SOCKS.

Installation
------------

`$ gem install socksify` *** NOTE *** Ruby < 3.1 presently

Usage
-----

### Redirect all TCP connections of a Ruby program

Run a Ruby script with redirected TCP through a local [Tor](https://web.archive.org/web/20170127085053/http://www.torproject.org/) anonymizer:

`$ socksify_ruby localhost 9050 script.rb`

### Explicit SOCKS usage in a Ruby program *** WARNING *** may be deprecated in future releases

Set up SOCKS connections for a local [Tor](https://www.torproject.org/) anonymizer, TCPSockets can be used as usual:

```
require 'socksify'

TCPSocket::socks_server = "127.0.0.1"
TCPSocket::socks_port = 9050
rubyforge_www = TCPSocket.new("rubyforge.org", 80)
# => #<TCPSocket:0x...>
```

### Use Net::HTTP explicitly via SOCKS

Require the additional library `socksify/http` and use the `Net::HTTP.SOCKSProxy` method. It is similar to `Net:HTTP.Proxy` from the Ruby standard library:
```
require 'socksify/http'

uri = URI.parse('http://ipecho.net/plain')
Net::HTTP.SOCKSProxy('127.0.0.1', 9050).start(uri.host, uri.port) do |http|
  req = Net::HTTP::Get.new uri
  resp = http.request(req)
  puts resp.inspect
  puts resp.body
end
# => #<Net::HTTPOK 200 OK readbody=true>
# => <A tor exit node ip address>
```
Note that `Net::HTTP.SOCKSProxy` never relies on `TCPSocket::socks_server`/`socks_port`. You should either set `SOCKSProxy` arguments explicitly or use `Net::HTTP` directly.

### Resolve addresses via SOCKS
```
Socksify::resolve("spaceboyz.net")
# => "87.106.131.203"
```
### Testing and Debugging

A tor proxy is required before running the tests. Install tor from your usual package manager, check it is running with `pidof tor` then run the tests with:

`ruby test/tc_socksify.rb`

Colorful diagnostic messages can be enabled via:

`Socksify::debug = true`

Development
-----------

The [repository](https://github.com/astro/socksify-ruby/) can be checked out with:

`$ git-clone git@github.com:astro/socksify-ruby.git`

Send patches via pull requests.

### Further ideas

*   `Resolv` replacement code, so that programs which resolve by themselves don't leak DNS queries
*   IPv6 address support
*   UDP as soon as [Tor](https://www.torproject.org/) supports it
*   Perhaps using standard exceptions for better compatibility when acting as a drop-in?

Author
------

*   [Stephan Maka](mailto:stephan@spaceboyz.net)

License
-------

SOCKSify Ruby is distributed under the terms of the GNU General Public License version 3 (see file `COPYING`) or the Ruby License (see file `LICENSE`) at your option.