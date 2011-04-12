# nolog.js ... a node.js based real time event driven log file watcher 

  - Author: Franz Enzenhofer http://twitter.com/enzenhofer
  - Thx: Tupalo.com for the awesome Hack Week <http://tupalo.com> 

nolog lets you watch (log)files in real time and matches regular expressions to events.

** New: now an npm package **

installation 

      npm install nolog

usage

      var $n = require("nolog");
      //or if you don't use npm
      //var $n = require("/path/to/nolog/nolog.js");
      
watch a file

      //$n(path)
      //returns a NologEventEmitter
      var log = $n("/path/to/the/logfile.log");
      
alternativly to `$n(path)` one can also write `$n.watch(path)`

note: to test the included examples localy you might want to funnel your webservers logfiles to a local test.log

    cd /path/to/nolog/
    ssh server.example.com sudo tail -f /var/log/nginx/example.com_access.log > test.log

shout (throw an event) if a single logfile line matches an reguar expression

      //NologEventEmitter.watch(eventname, pattern)
      log.shoutIf('googlebot', /Googlebot/i );
      log.shoutIf('googlebot', "Chrome" );

the `pattern` argument can either be a regular expression or a simple string ( means .*+?|()[]{}\ will be treated as 'normal' characters)



`shoutIfNot` is the same (but opposite) of `shoutIf` - `shoutIfNot` throws an event if the pattern did not lead to a successfull match.

      //NologEventEmitter.shoutIfNot(eventname, pattern)
      log.shoutIfNot('allNotGetRequests', "GET" );
      
`shoutIf` and `shotIfNot` start so called shout-jobs. 

`si` and `sin` are shortcuts for `shoutIf` and `shoutIfNot`
      
assign an event handler
      
      //NologEventEmitter.on(eventname, callback)
      log.on('googlebot', function(data){ console.log("a Googlebot");});
      
`on` assigns an event handler to the `NologEventEmitter` listening for the `eventname`.

`shoutIf` and `shoutIfNot` assigned events return an RegExp return array <https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/RegExp/exec#Description> that's an array with a special `index` and `input` value.

as there was nothing found at `shouldIfNot` the `index` value is null, the first value of the array is also `null`. `input` is the parsed log line.


for other standard methods of NologEventEmitter see EventEmitter in the offical Node.js documentation <http://nodejs.org/docs/v0.4.5/api/events.html#events.EventEmitter>

kill a `shoutIf` or `shoutIfNot` job
      
      //NologEventEmitter.kill(eventname)
      log.kill('googlebot');

no more `googlebot` events will be thrown. the shout-job with the name `eventname` gets killed.

`unwatch` a file

      //NologEventEmitter.unwatch()
      log.unwatch();
      
`unwatch` kills all shout-jobs (no more events get thrown) and stops the background task watching the specific file.

`killAll` is a synonym for `unwatch`

if you `kill` the last last shout-job of a watched file, the file is automatically `unwatch`ed. said that, the `NologEventEmitter` object is still valid, if you add a new shout-job, the file is automatically re-`watch`ed. (think: magic)

**MAGIC STUFF**

`shout` is fluffy shortcut for `shoutIf`
      
      //NologEventEmitter.shout(eventname)
      log.shout('googlebot'); // == log.shout('googlebot', /googlebot/i );
      
basically it `shout` throws an event with the name `eventname` if a case-iNsEnSeTiVE match for `eventname` was found.


**USEFULL STUFF**

 - all public methods support chaining (i.e.$n('access.log').s('bot').on('bot', function(d){ ... }); 
 - `nolog.enableDebug(true)` - turns on some debug output via `console.log`
 
 - `NologEventEmitter.file` holds the watched `file` string, default `null`
 - `NologEventEmitter.follow` enables real time updates, default `true`
 - `NologEventEmitter.wholeFile` parses the whole file from the beginning, default `false` (note: handle with care, can fire millions of events at once, not suitable for browser clients applications)
 - `NologEventEmitter.jobs` array that holds a summary of active shout-jobs assigned ot the currently watched file
 
 the `follow` and `wholefile` attribut can be also passed via the `watch` method.
      
      var log = nolog.watch("/path/to/the/logfile.log", {wholefile:true; follow:false});
      

 
 
**ADVANCED STUFF**

notEvents

      //NologEventEmitter.shoutIf(eventname,pattern,enableNotEvent)
      log.shoutIf('bot', /bot/i, true );

additonally to the success event 'bot' another event is thrown, namely '!bot' if the pattern was not matched. this means, for evey parsed logline an event -either `eventname` or `!eventname` - is thrown.

this is not the same as `shoutIfNot` - `shoutIfNot` throws a success match, if something was not matched. as a matter of fact, `shoutIfNot` also supports `enableNotEvent`
      
      //NologEventEmitter.shoutIfNot(eventname,pattern,enableNotEvent)
      log.shoutIfNot('human', /bot/i, true );

if the logline does not match the '/bot/i' the event 'human' is thrown. if the logline matches the event '!human' is thrown. (note: notEvents are less confusing when used with `shoutIf`)


boolean patterns

`si` and `sin` (and their longer worded aliases `shoutIf` and `shoutIfNot`) also support booleans patterns.
      
      //get every line
      log.si('line', true).on('line', function(data){ ... });

function callback patterns (that feature is alpha)

`si` and `sin` can also handle function patterns.

**TODO**
  - enable streams other than files to get imported
  - native ssh support?












